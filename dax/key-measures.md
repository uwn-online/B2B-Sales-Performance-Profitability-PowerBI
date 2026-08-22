# Key DAX Measures

This document presents selected DAX measures used in the B2B Sales Performance
& Profitability dashboard. These measures support executive performance analysis,
profitability monitoring, target achievement, commercial growth analysis and
commercial-risk reporting.

## 1. Gross Sales

Calculates total sales value after transaction-level discounts but before deducting
customer returns.

```DAX
Gross Sales =
SUM(
    'Sales Transactions'[Sales Amount]
)
```

## 2. Returned Sales

Calculates the monetary value of sales subsequently returned by customers.

```DAX
Returned Sales =
SUM(
    'Sales Transactions'[Returned Amount]
)
```

## 3. Net Sales

Calculates retained sales revenue after deducting returned sales from Gross Sales.

```DAX
Net Sales =
[Gross Sales] - [Returned Sales]
```

## 4. Gross Profit

Calculates total Gross Profit generated within the current reporting context.

```DAX
Gross Profit =
SUM(
    'Sales Transactions'[Gross Profit]
)
```

## 5. Gross Margin %

Calculates Gross Profit as a percentage of Net Sales. The calculation uses
aggregate Gross Profit and Net Sales rather than averaging row-level margin percentages.

```DAX
Gross Margin % =
DIVIDE(
    [Gross Profit],
    [Net Sales],
    0
)
```

## 6. Sales Target

Calculates the total sales target for the current reporting context.

```DAX
Sales Target =
SUM(
    'Sales Transactions'[Target Sales Amount]
)
```

## 7. Target Achievement %

Measures the percentage of the sales target achieved through Net Sales.

```DAX
Target Achievement % =
DIVIDE(
    [Net Sales],
    [Sales Target],
    0
)
```

## 8. Target Shortfall

Calculates the monetary gap between the Sales Target and actual Net Sales where
the business has not achieved plan.

```DAX
Target Shortfall =
MAX(
    0,
    [Sales Target] - [Net Sales]
)
```

## 9. Return Rate %

Calculates the percentage of pre-return sales value subsequently lost through
customer returns.

```DAX
Return Rate % =
DIVIDE(
    [Returned Sales],
    [Gross Sales],
    0
)
```

## 10. Order Count

Counts unique customer orders rather than individual sales lines.

```DAX
Order Count =
DISTINCTCOUNT(
    'Sales Transactions'[Order ID]
)
```

## 11. Average Order Value

Calculates average Net Sales generated per distinct customer order.

```DAX
AOV =
DIVIDE(
    [Net Sales],
    [Order Count],
    0
)
```

## 12. List Sales

Calculates the potential sales value before transaction-level discounts are applied.

```DAX
List Sales =
SUMX(
    'Sales Transactions',
    'Sales Transactions'[Quantity]
        * 'Sales Transactions'[List Unit Price]
)
```

## 13. Discount Cost

Calculates the monetary value of revenue surrendered through discounting.

```DAX
Discount Cost $ =
[List Sales] - [Gross Sales]
```

## 14. Effective Discount Rate

Calculates Discount Cost as a percentage of total List Sales. This provides a
value-weighted view of discount intensity rather than simply averaging individual
discount percentages.

```DAX
Effective Discount Rate =
DIVIDE(
    [Discount Cost $],
    [List Sales],
    0
)
```

## 15. Business Event Revenue

Calculates Net Sales associated with defined exceptional business events while
excluding transactions classified as Normal Trading.

```DAX
Business Event Revenue =
CALCULATE(
    [Net Sales],
    'Business Event'[Business Event] <> "Normal Trading"
)
```

## 16. Business Event Revenue Share %

Calculates the percentage of total Net Sales associated with exceptional business
events.

```DAX
Business Event Revenue Share % =
DIVIDE(
    [Business Event Revenue],
    [Net Sales],
    0
)
```

## 17. Shipping Cost

Calculates total shipping and fulfilment expenditure.

```DAX
Shipping Cost =
SUM(
    'Sales Transactions'[Shipping Cost]
)
```

## 18. Average Order-to-Ship Days

Calculates average fulfilment time at distinct-order level while excluding
cancelled orders.

```DAX
Avg. Order-to-Ship Days =
AVERAGEX(
    FILTER(
        VALUES('Sales Transactions'[Order ID]),
        CALCULATE(
            MAX('Sales Transactions'[Order Status])
        ) <> "Cancelled"
    ),

    VAR OrderDateKey =
        CALCULATE(
            MIN('Sales Transactions'[Order Date Key])
        )

    VAR ShipDateKey =
        CALCULATE(
            MIN('Sales Transactions'[Ship Date Key])
        )

    VAR OrderDate =
        DATE(
            INT(OrderDateKey / 10000),
            INT(MOD(OrderDateKey, 10000) / 100),
            MOD(OrderDateKey, 100)
        )

    VAR ShipDate =
        DATE(
            INT(ShipDateKey / 10000),
            INT(MOD(ShipDateKey, 10000) / 100),
            MOD(ShipDateKey, 100)
        )

    RETURN
        DATEDIFF(
            OrderDate,
            ShipDate,
            DAY
        )
)
```

## Measure Design Considerations

- The Sales Transactions table operates at sales-line grain, so order-level
  calculations use distinct Order IDs rather than row counts.
- Net Sales is calculated after deducting Returned Sales.
- Gross Margin is calculated from aggregate Gross Profit and Net Sales rather
  than averaging row-level percentages.
- Return Rate is value-based and measures Returned Sales as a percentage of
  Gross Sales.
- Effective Discount Rate is weighted by sales value through List Sales rather
  than calculated using a simple average of Discount Pct.
- Business Event Revenue identifies revenue associated with defined business
  events and should not be interpreted as proof of causation.
- Cancelled orders are excluded from the Average Order-to-Ship Days calculation.
- Measures respond dynamically to report filters, year selections, drill-down
  selections and cross-chart interactions.
