---
title: Updating data
sidebar_label: Updating data
description:
  How the UPDATE statement is implemented in QuestDB.
---

This document describes how the UPDATE statement works in QuestDB and what happens
under the hood when an update is issued.

## Storage model

To be able to understand how rows of a table is updated in QuestDB, first we
need to have an idea of how the data is stored. The documentation contains
detailed descriptions of the [storage model](/docs/concept/storage-model) and
the [directory layout](/docs/concept/root-directory-structure#db-directory)
but if we quickly want to summarize it:
- Each table has its own folder in the db root, the directory is named after the table
- Partitions are manifested as subdirectories under the folder represents the table
- The actual data is stored in column files inside these subdirectories
- Column files store data **ordered by the designated timestamp** and they are
**append-only**. This goes naturally with time series data, just think about market
data where the price of different financial instruments are tracked during the
trading day, for example

## Column versions

Since the data is stored in order and the files are append-only we are in a bit of
trouble with updating it. We took the optimistic approach and assumed that past data
will never have to change. This is great, helps a lot to optimize performance.
However, sometimes we need to **amend data** which was recorded incorrectly because of a
bug or for any other reason.

We could break our append-only model and start accessing different parts of the
column files to fix incorrect data. The problem we would face with is inconsistent
reads. Readers running queries on the table would not be happy as they could see
some part of the data updated but not others.

The solution is to make the update **transactional** and **copy-on-write**. Basically
a new column file is created when processing the UPDATE statement. All readers are
looking at a previous consistent view of the data from an older column file while the
UPDATE is in progress. Readers can find the latest committed version of the column files
based on a table stored in a metadata file. When the update is completed and a new
column version is available for the readers, the metadata gets updated as part of
the commit. After the metadata change new SELECT queries will see the updated data.

## Limitations

Current implementation of the UPDATE operation rewrites the column files by copying
records in their existing order from the previous version, and replacing the value if
it needs changing. As a result the **designated timestamp cannot be updated.**

Modifying the designated timestamp would lead to rewriting history of the time series.
Records would need to be reordered, this could even mean moving rows in between
different partitions. Not impossible to do but definitely more challenging, something
for the future to think about.
