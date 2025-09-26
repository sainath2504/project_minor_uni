
# E‑commerce Sales Dashboard (SQL + Power BI)

This folder contains a **portfolio‑ready** dataset and SQL schema for building an end‑to‑end E‑commerce Sales Dashboard with **Orders, Order Items, Products, Customers, Returns**, and a **Calendar** table.

## Files
- `customers.csv` – 800 customers across Canada/USA with regions and signup dates
- `products.csv` – 250 products with category, brand, unit cost/price
- `orders.csv` – 3,500 orders over the last 12 months with channel, payment method, status
- `order_items.csv` – line‑level detail with quantities, discounts, revenue, and cost
- `returns.csv` – item‑level returns for ~8% of orders
- `calendar.csv` – date table for time intelligence
- `schema.sql` – SQL DDL and a sample KPI query (Revenue/COGS/Gross Margin by month)

## Suggested Power BI Model
1. Import all CSVs into Power BI Desktop (Get Data → Text/CSV).
2. Set relationships:
   - `orders[order_id]` 1‑* `order_items[order_id]`
   - `products[product_id]` 1‑* `order_items[product_id]`
   - `customers[customer_id]` 1‑* `orders[customer_id]`
   - `calendar[date]` 1‑* `orders[order_date]` (mark `calendar` as Date Table)
   - (Optional) Create a returns bridge: relate `returns` to `order_items` on (`order_id`,`product_id`) and use it in measures.
3. Ensure `order_date` is a true Date data type.

## Core DAX Measures (copy into a Measures table)
```
Total Revenue =
SUM ( order_items[line_total] )

Total COGS =
SUM ( order_items[line_cost] )

Gross Margin =
[Total Revenue] - [Total COGS]

Gross Margin % =
DIVIDE ( [Gross Margin], [Total Revenue] )

Total Orders =
DISTINCTCOUNT ( orders[order_id] )

AOV =
DIVIDE ( [Total Revenue], [Total Orders] )

Items per Order =
DIVIDE ( SUM(order_items[quantity]), [Total Orders] )

Returns Qty =
SUM ( returns[quantity_returned] )

Returns Rate (by Items) =
DIVIDE ( [Returns Qty], SUM(order_items[quantity]) )

Net Revenue (after returns) =
VAR Rev = [Total Revenue]
VAR ReturnedValue =
SUMX (
    returns,
    returns[quantity_returned] *
    CALCULATE ( AVERAGE ( order_items[unit_price] ), TREATAS ( VALUES(returns[product_id]), order_items[product_id] ) )
)
RETURN Rev - ReturnedValue
```

## Recommended Visuals
- **KPI Cards:** Total Revenue, Gross Margin %, Total Orders, AOV
- **Time Series:** Revenue by Month (calendar[month_name] or Month hierarchy)
- **Bar Charts:** Revenue by Category, Brand, Region, Sales Channel
- **Tree Map:** Revenue by Product
- **Table:** Top 20 Products (Revenue, Qty, GM%)
- **Scatter:** AOV by Region vs Orders
- **Slicer:** Date, Category, Channel, Country

## Sample SQL (Monthly Revenue/COGS/Margin)
See `schema.sql` for a ready‑to‑run query (Postgres‑style). Adjust date truncation for your SQL dialect.
