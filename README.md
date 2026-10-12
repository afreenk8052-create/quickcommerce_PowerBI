Quick-Commerce Dark Store Operations & Delivery Performance Dashboard

An individual Power BI project analysing delivery speed, delays, revenue and customer ratings across 31 dark stores in 8 Indian cities.

Author: [Your Name] | Course: [Course / Batch Name] | Submitted: October 2026

📌 Table of Contents
Project Overview
Business Problem & Objectives
Dataset
Tools Used
Data Model
DAX Measures
Dashboard Pages
Key Insights
Recommendations
Repository Structure
How to Run
Limitations & Future Work
📖 Project Overview

Quick-commerce companies (Blinkit, Zepto, Swiggy Instamart and similar) promise delivery in about 20 minutes from small local warehouses called dark stores. This project builds an interactive Power BI dashboard that helps operations managers see where and when those promises are missed, and what it costs in revenue and customer satisfaction.

🎯 Business Problem & Objectives

Problem: Delayed deliveries reduce customer ratings and repeat orders, but management cannot easily see which cities, stores or time slots cause the delays.

Objectives

Track core KPIs: orders, net revenue, average order value, on-time %, cancellation % and average rating.
Identify the cities and stores with the weakest delivery performance.
Find the peak hours when delays rise.
Measure how delays affect customer ratings.
Understand revenue by category, city and customer segment, and the effect of discounts.
📂 Dataset

⚠️ Note: The dataset is synthetic, generated with Python (scripts/generate_dataset.py) for academic purposes. It is modelled on typical quick-commerce operations in India and is not real company data.

File	Rows	Description
data/quickcommerce_orders.csv	8,000	One row per order (Jan–Sep 2026)
data/dark_stores.csv	31	One row per dark store

Orders table columns

Column	Description
Order_ID, Order_Date, Order_Hour	Order identifier and timing
Store_ID, City	Dark store that fulfilled the order and its city
Category	Product category (8 categories)
Customer_Segment	New / Regular / Premium
Payment_Mode	UPI / Card / Cash on Delivery / Wallet
Items_Count	Number of items in the order
Order_Value, Discount, Delivery_Fee, Net_Revenue	Money columns in ₹ (Net_Revenue is 0 for cancelled orders)
Prep_Time_Min, Travel_Time_Min, Delivery_Time_Min	Time to pack, time to travel, and total
Promised_Time_Min	Delivery promise (20 min, or 25 min for Pharmacy and Electronics)
Delivery_Status	On Time / Delayed / Cancelled
Rating	Customer rating, 1–5 (blank for cancelled orders, intentionally)
Delivery_Partner_ID	Rider identifier

Stores table columns: Store_ID, City, Area, Capacity_Orders_Per_Day, Staff_Count, Opened_Year.

🛠 Tools Used
Microsoft Power BI Desktop: data modelling, DAX and dashboards
Power Query: data cleaning and type conversion
Python (pandas, NumPy, Matplotlib): dataset generation and preview charts
GitHub: version control and submission
🧩 Data Model

A simple star schema:

dark_stores (Store_ID) 1 ───< * orders (Store_ID) *>─── 1 DateTable (Date)
orders is the fact table; dark_stores and DateTable are dimension tables.
A custom DateTable is created with DAX, and Month and Weekday are sorted using their number columns.
🧮 DAX Measures
DAX
Total Orders      = COUNTROWS(orders)
Delivered Orders  = CALCULATE([Total Orders], orders[Delivery_Status] <> "Cancelled")
Total Revenue     = SUM(orders[Net_Revenue])
Avg Order Value   = DIVIDE([Total Revenue], [Delivered Orders])
Discount %        = DIVIDE(SUM(orders[Discount]), SUM(orders[Order_Value]))
Avg Delivery Time = CALCULATE(AVERAGE(orders[Delivery_Time_Min]), orders[Delivery_Status] <> "Cancelled")
On-Time %         = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "On Time"), [Total Orders])
Delay %           = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "Delayed"), [Total Orders])
Cancellation %    = DIVIDE(CALCULATE([Total Orders], orders[Delivery_Status] = "Cancelled"), [Total Orders])
Avg Rating        = AVERAGE(orders[Rating])
Revenue MoM %     = VAR cur  = [Total Revenue]
                    VAR prev = CALCULATE([Total Revenue], DATEADD(DateTable[Date], -1, MONTH))
                    RETURN DIVIDE(cur - prev, prev)

🔍 Key Insights
KPI	Value
Total orders	8,000
Net revenue	₹33.4 lakh
Average order value	₹428
On-time delivery	75.2%
Delayed / Cancelled	22.2% / 2.6%
Average rating	4.14
Delays follow meal-time peaks. Delay % jumps to about 36–37% at lunch (12:00–13:00) and dinner (19:00–21:00), compared with roughly 11–16% in off-peak hours.
Delays hurt ratings. Delayed orders average 3.28 stars against 4.40 for on-time orders, a drop of more than one star.
Bengaluru and Mumbai are the slowest. Bengaluru averages 18.5 minutes and Mumbai 18.1, while Prayagraj averages 14.9. Bengaluru's delay % stays around 30–35% on every weekday.
Revenue is concentrated. Mumbai (₹7.4L) and Delhi (₹6.1L) lead the cities; Groceries (₹7.0L) and Electronics (₹6.9L) lead the categories. Electronics has by far the highest average order value (about ₹1,768).
New customers get the biggest discounts. Discounts are 10.7% of order value for New, 6.2% for Regular and 4.5% for Premium customers.
Regular customers drive volume: 54% of orders, with 26% New and 20% Premium. UPI is the most used payment mode at 56%.
