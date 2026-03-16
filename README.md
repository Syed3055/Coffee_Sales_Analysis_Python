# 🍵 Coffee Sales Analysis — Power BI Report README
##### (STAR-format: Situation → Task → Actions → Results )

# 📌 Executive Summary

Total Sales: ₹ 122,321.58 across 3,898 orders; Avg. Order Value: ₹ 31.38.
Payment Mix (by revenue): Card ₹117,114.58 (95.7%), Cash ₹5,207.00 (4.3%). By order count: Card 3,729, Cash 169.
Top months by sales: Feb‑2025 ₹16,804.48, Oct‑2024 ₹13,891.16, Mar‑2025 ₹13,287.44.
Orders by month (across all years combined): Mar (649), Feb (561), Oct (426) are the busiest; Jan (201) & Apr (196) are slow.
Top items by sales share: Latte 23.43%, Americano with Milk 20.66%, Cappuccino 15.14%, Americano 12.62%, Hot Chocolate 8.32% (≈80% combined).


#### Data sources: Two CSVs were appended into a single Master fact table in Power BI (Power Query). [index_1 | Excel], [index_2 | Excel]


## S — Situation
You had two CSV exports:

index_1.csv: transactions (2024) with cash_type, card, money, coffee_name, and timestamp fields.
index_2.csv: subsequent transactions (2025) with the same core fields, but without the card column. [index_1 | Excel], [index_2 | Excel]

Business need: Create a single analytical model and an interactive Power BI report to monitor Total Sales, Orders, MoM growth, %MoM, YTD, SPLY, Payment mix (Card vs Cash), and % Item Sales, then interpret performance and risks.

## T — Task

Model: Unify both CSVs into one Master table; reconcile missing columns; standardize data types.
Calendar: Build a robust Date dimension with CALENDARAUTO() and derived time columns (Year, Quarter, Month, MonthName, Year_Month, Day, Day_Name).
Measures: Create core DAX measures: Total Sales, Total Orders, MTD, MoM, %MoM, YTD, SPLY, Card Sales, % Card Sales, % Item Sales.
Report: Deliver two pages:

### Overview: KPIs, monthly Orders column chart, Sales line by Year–Quarter–Month, payment/item summary, and slicers.
Detail View: Drillable matrix (Year→Quarter→Month) with sales trend measures and an item share panel.


Insights: Read the visuals and articulate what the numbers tell us + risks/mitigations.


## A — Actions
#### 1) Data ingestion & shaping (Power Query)

Append the two CSVs to form Master:

Ensured schema alignment (added empty card column to the 2025 file), fixed data types for date, datetime, money, and text columns.


Cleansing: Normalized item names (“Americano with Milk” vs casing), removed obvious duplicates, and validated nulls in cash_type (defaulted to “card” or “cash” per source).
Calculated column alignment: Ensured both files expose a consistent cash_type field so payment-split measures work seamlessly.
(All above based on the actual structure seen in both CSVs.) [index_1 | Excel], [index_2 | Excel]

## 2) Date dimension (DAX)
DAXCalendar = CALENDARAUTO()-- Mark this as Date Table (use Calendar[Date]).Year        = YEAR('Calendar'[Date])Quarter     = "Q" & FORMAT('Calendar'[Date], "Q")Month No    = MONTH('Calendar'[Date])Month Name  = FORMAT('Calendar'[Date], "MMMM")Year-Month  = FORMAT('Calendar'[Date], "YYYY-MM")Day         = DAY('Calendar'[Date])Day Name    = FORMAT('Calendar'[Date], "DDDD")Show more lines



## 3) Data model

### Relationship: Calendar[Date] (1) → Master[date] (∗).
Ensure single-direction filter from Calendar to Master (typical Star schema).
Disable “Auto Date/Time” to avoid implicit date tables, keeping your explicit Calendar clean.

## 4) Measures (DAX)
DAX-- CoreTotal Sales   = SUM(Master[money])Total Orders  = COUNTROWS(Master)   -- one row per order in this dataset-- Payment splitCard Sales    = CALCULATE([Total Sales], Master[cash_type] = "card")Cash Sales    = CALCULATE([Total Sales], Master[cash_type] = "cash")% Card Sales  = DIVIDE([Card Sales], [Total Sales])-- Time intelligenceSales LY      = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))Sales MTD     = TOTALMTD([Total Sales], 'Calendar'[Date])Sales YTD     = TOTALYTD([Total Sales], 'Calendar'[Date])Sales PM      = CALCULATE([Total Sales], DATEADD('Calendar'[Date], -1, MONTH))MoM           = [Total Sales] - [Sales PM]% MoM         = DIVIDE([MoM], [Sales PM])-- Item contribution (use in item context)% Item Sales  = DIVIDE([Total Sales], CALCULATE([Total Sales], ALL(Master[coffee_name])))Show more lines
## 5) Visual build (as seen in the two screenshots)
#### Page 1 — Overview


#### KPI Cards: Sales ₹122.32K and Orders 3.90K + mini-cards for Cash and Card sales.
Total Orders by Month (clustered columns): Month name on X, Orders as value. (You grouped across all years; that’s why March=649 orders, Feb=561, Oct=426, Jan=201, Apr=196, etc.)
Payment vs Item summary (table): Item-wise orders & sales; visible dominance of Latte, Americano with Milk, Cappuccino.
Total Sales by Year, Quarter, Month (line): Shows the build-up across 2024 into early 2025 with spikes around Oct‑2024, Feb‑2025, and Mar‑2025.
Slicers: From–To Date, Q1–Q4 buttons, Year buttons (2024/2025), and dropdowns for Item Name, Payment Type, Month Name.
Theme: Coffee palette (latte/beige, cocoa/brown), rounded cards, and reset button for bookmarks.

### Page 2 — Detail View


Matrix (Year → Quarter → Month): Columns show Total Sales, Pre. Month Sales, MTD, %MoM, YTD, SPLY with red/green conditional formatting on %MoM.
Right panel (Item share): Item list ranked by Total_sales with % Item Share; top 5 items ≈80% of revenue.


## R — Results, Insights & Recommendations
What the report shows (facts)

Overall performance (all time): ₹ 122,321.58 from 3,898 orders; AOV ₹31.38.
Customer payment behavior: Card accounts for 95.7% of sales (₹117.11K) and 95.7% of orders (3,729/3,898), making revenue highly dependent on card acceptance.
Seasonality / demand pattern:

Busiest months by orders (across all years): Mar (649), Feb (561), Oct (426). Slow months: Jan (201), Apr (196).
Top months by sales (year–month): Feb‑2025 (₹16.80K), Oct‑2024 (₹13.89K), Mar‑2025 (₹13.29K).
MoM, early‑2025: Jan‑2025 fell ‑22.3% vs Dec‑2024; Feb‑2025 rebounded +162.6%; Mar‑2025 softened −20.9% vs Feb.


## Product mix:

Latte (23.43%) and Americano with Milk (20.66%) are the key drivers, followed by Cappuccino (15.14%), Americano (12.62%), Hot Chocolate (8.32%). Focused assortment accounts for ≈80% of sales.



### What this means: 

Sales concentration risk: A few items dominate revenue; any supply/pricing issues on Latte or Americano with Milk will disproportionately impact topline.
Operational exposure to card systems: With >95% of revenue via card, any POS/acquirer outage could materially reduce sales during impacted hours/days.
Seasonal opportunities: Feb–Mar and Oct are demand peaks. Capitalize with bundles, limited-time flavors, or upsell (e.g., add‑on pastries) in those months.
Demand gaps: Jan/Apr underperform—plan promotions, loyalty campaigns, or price experimentation to lift traffic.

Mitigate the risks (actionable)

### Product risk:

Build second‑tier heroes (e.g., Cappuccino, Americano, Hot Chocolate) via bundles or loyalty points to reduce reliance on Latte/Americano‑Milk.
Monitor stock & COGS for top items weekly; pre‑order critical SKUs before peaks.


### Payment dependency:

Ensure dual connectivity (primary broadband + 4G failover) and backup QR/UPI rails for downtime.
Track daily card vs cash conversion; promote cash/UPI discounts during outages to retain throughput.


### Seasonality playbook:

Peak months: Pre‑staffing, inventory build, LTOs, social ads.
Slow months: Targeted offers, happy‑hour pricing, combo deals, and cross‑sell (desserts).


### Analytics hygiene:

Keep Calendar marked as Date Table and sort Month Name by Month No to prevent time‑intelligence misalignment.
Add Item Category (e.g., Coffee / Chocolate / Tea) and Hour of Day for deeper insights (barista staffing, batch prep).
Add a simple Returns/Voids flag if applicable to net‑out sales correctly.




## How the visuals were created (step‑by‑step)

Below is a compact recipe you (already) followed, written so anyone can reproduce the report from scratch.


### Load data: Get data → Text/CSV twice → Append queries as Master. Add missing card column to the second file; set proper data types. [index_1 | Excel], [index_2 | Excel]
Create Calendar: Modeling → New table → Calendar = CALENDARAUTO(). Add Year/Quarter/Month/Day columns; mark as Date table.
Relate: Create one‑to‑many from Calendar[Date] to Master[date].
Measures: Add DAX for Total Sales, Total Orders, Card/Cash Sales, %Card, YTD, SPLY, MoM, %MoM, MTD, % Item Sales (formulas provided above).
Overview page:

### Cards: [Total Sales], [Total Orders], plus cards for [Cash Sales] & [Card Sales].
Column chart (Orders by Month): Axis Calendar[Month Name] (sorted by Month No), value Total Orders. (This is why the chart shows Mar=649, Feb=561, Oct=426…; it aggregates months across years.)
Line chart (Sales by Y-Q-M): Axis Calendar[Year-Month], value [Total Sales]. Use data labels for peaks (Feb‑2025 ~₹16.8K, Oct‑2024 ~₹13.9K, Mar‑2025 ~₹13.3K).
Table (Item + Payment): Rows Master[coffee_name], values [Total Orders], [Total Sales] (optionally show split by cash_type).
Slicers: Date range, Quarter buttons (Q1–Q4), Year buttons (2024/2025), plus Item Name, Payment Type, Month Name.


## Detail View page:

Matrix with Year → Quarter → Month; values: [Total Sales], [Sales PM], [Sales MTD], %MoM, [Sales YTD], [Sales LY]. Use field formatting to color %MoM red (negative) and green (positive).
Right‑side Item share panel: table of Item Name, [Total Sales], % Item Sales (descending).


## UX polish:

Coffee‑themed color palette (cream/brown), rounded cards, subtle gridlines.
“Reset” button using Bookmarks to clear slicers.




## Validation snapshots (from the model)

Grand totals: Sales ₹122,321.58, Orders 3,898 (report cards).
Payment split: Card ₹117,114.58, Cash ₹5,207.00; Orders: Card 3,729, Cash 169.
Month aggregation (all years combined):
Jan 201, Feb 561, Mar 649, Apr 196, May 267, Jun 227, Jul 237, Aug 272, Sep 344, Oct 426, Nov 259, Dec 259 (mirrors your bar chart).
Top 5 items by sales share: Latte 23.43%, Americano with Milk 20.66%, Cappuccino 15.14%, Americano 12.62%, Hot Chocolate 8.32%.
Time‑series highlights: Feb‑2025 ₹16.8K (MoM +162.6%), Mar‑2025 ₹13.29K (MoM −20.9%); Mar‑2025 vs Mar‑2024 (SPLY): ₹13.29K vs ₹7.05K.


## Assumptions & Notes

Each row ≈ one order line → COUNTROWS is a good proxy for Total Orders.
Currency assumed consistent (₹). If multi‑currency data is added later, introduce a Currency column and a conversion table.
Data coverage starts Mar‑2024, so SPLY for Jan–Feb 2025 is naturally 0.
Source files: index_1.csv (2024 base), index_2.csv (2025 extension). [index_1 | Excel], [index_2 | Excel]


## Next Improvements (quick wins)

Add Hierarchies: Item → Category to see category‑level trends.
Customer/Channel fields: If available (e.g., Store, Region, or Session), unlock segmentation and cohort retention.
Time of Day trends: Add Hour from datetime to optimize staffing and promos.
Price tracking: A price reference table to detect discounting vs list price impacts on AOV.
Quality gates: Power Query data quality steps (outliers, nulls, schema validation) + refresh failure alerts
#### Author: Syed........
####  ---------- Thank You ................💐💐💐
