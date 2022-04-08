---
title: UPDATE keyword
sidebar_label: UPDATE
description: UPDATE SQL keyword reference documentation.
---

Updates data in a database table.

## Syntax

![Flow chart showing the syntax of the UPDATE keyword](/img/docs/diagrams/update.svg)

:::note

- the same `columnName` cannot be specified more times after the SET keyword as it would be ambiguous
- the designated timestamp column cannot be updated as it would lead to altering history of the time series data

:::

## Examples

```questdb-sql title="Update with constant"
UPDATE trades SET price = 125.34 WHERE symbol = 'AAPL';
```

```questdb-sql title="Update with function"
UPDATE book SET mid = (bid + ask)/2 WHERE symbol = 'AAPL';
```

```questdb-sql title="Update with join"
UPDATE spreads s SET s.spread = p.ask - p.bid FROM prices p WHERE s.symbol = p.symbol;
```
