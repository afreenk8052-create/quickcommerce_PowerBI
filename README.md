## 1. Business Problem
Quick-commerce companies promise delivery in 15-25 minutes. Management needs to know:
- Which cities and stores miss their delivery promise, and why?
- When (hour of day) do delays peak?
- Which categories and customer segments drive revenue?
- How do delays affect customer ratings?

## 2. Dataset
| File | Rows | Description |
|---|---|---|
| quickcommerce_orders.csv | 8,000 | Fact table: one row per order |
| dark_stores.csv | 31 | Dimension table: store details |

Key columns in orders: Order_ID, Order_Date, Order_Hour, Store_ID, City, Category, Customer_Segment, Payment_Mode, Items_Count, Order_Value, Discount, Delivery_Fee, Net_Revenue, Prep_Time_Min, Travel_Time_Min, Delivery_Time_Min, Promised_Time_Min, Delivery_Status, Rating, Delivery_Partner_ID.
(Rating is blank for cancelled orders - this is intentional.)

## 3. Load & Model (Power BI Desktop)
1. Home > Get Data > Text/CSV > load both files > **Transform Data**.
2. In Power Query: set Order_Date to *Date*, numeric columns to *Decimal Number*, Order_Hour and Items_Count to *Whole Number*. Check that Rating is left blank (not 0) for cancelled orders. Close & Apply.
3. Model view: link `dark_stores[Store_ID]` (1) -> `orders[Store_ID]` (many), single direction.
4. Create a Date table (Modeling > New Table):
```DAX
DateTable = ADDCOLUMNS(CALENDAR(DATE(2026,1,1), DATE(2026,12,31)),
    "Year", YEAR([Date]), "MonthNo", MONTH([Date]),
    "Month", FORMAT([Date], "MMM"), "Weekday", FORMAT([Date], "ddd"),
    "WeekdayNo", WEEKDAY([Date], 2))
```

## 4. DAX Measures
```DAX
Total Orders = COUNTROWS(orders)
Delivered Orders = CALCULATE([Total Orders], orders[Delivery_Status] <> "Cancelled")
Total Revenue = SUM(orders[Net_Revenue])
Avg Order Value = DIVIDE([Total Revenue], [Delivered Orders])
Total Discount = SUM(orders[Discount])
Discount % = DIVIDE([Total Discount], SUM(orders[Order_Value]))
Avg Delivery Time = CALCULATE(AVERAGE(orders[Delivery_Time_Min]), orders[Delivery_Status] <> "Cancelled")
On-Time % = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "On Time"), [Total Orders])
Delay % = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "Delayed"), [Total Orders])
Cancellation % = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "Cancelled"), [Total Orders])
Avg Rating = AVERAGE(orders[Rating])
Revenue MoM % = VAR cur = [Total Revenue]
    VAR prev = CALCULATE([Total Revenue], DATEADD(DateTable[Date], -1, MONTH))
    RETURN DIVIDE(cur - prev, prev)
Store Utilisation % = DIVIDE([Total Orders] / 273, SUM(dark_stores[Capacity_Orders_Per_Day]))
```

## 5. Dashboard Layout (3 pages)

**Page 1 - Executive Overview**
- KPI cards: Total Orders, Total Revenue, Avg Order Value, On-Time %, Avg Rating, Cancellation %
- Line chart: Total Revenue by Month
- Clustered bar: Revenue by City
- Donut: Revenue by Category
- Column chart: Orders by Customer Segment
- Slicers: Month, City, Category

**Page 2 - Delivery Performance**
- Gauge: On-Time % (target 80%)
- Bar chart: Avg Delivery Time by City
- Column chart: Orders and Delay % by Order_Hour (use Line and clustered column chart)
- Heat-map matrix: City (rows) x Weekday (columns), value = Delay %, with conditional formatting
- Scatter: Prep time vs Travel time per Store
- Table: Top 10 stores with highest Delay %

**Page 3 - Customer & Revenue Insights**
- Clustered column: Avg Rating by Delivery_Status
- Stacked bar: Payment_Mode by Customer_Segment
- Treemap: Revenue by Category
- Bar: Discount % by Customer_Segment
- Decomposition tree: Delay % broken by City > Store > Hour (AI visual)
- Map: Orders by City (filled map).

## 6. Insights to Mention in Your PPT
(Check these against your own visuals before presenting - read the actual numbers from your dashboard.)
- Delays rise during lunch (12-13h) and dinner (19-21h) peaks because prep time increases.
- Metro cities (Bengaluru, Mumbai) have the longest average delivery times; Prayagraj has the shortest.
- Delayed orders get noticeably lower ratings than on-time orders.
- New customers get the highest discount rate.
- Groceries and Fruits & Vegetables lead in order volume; Electronics has the highest order value.
