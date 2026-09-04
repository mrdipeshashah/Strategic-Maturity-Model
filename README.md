# STRATEGIC MATURITY MODEL

## OVERVIEW
Focusing on DTC/E-commerce business generating up to £10m in revenue I have developed a process using the Google Ecoysystem to better understand LTV which is supported by a robust set of metrics. It takes DTC/E-commerce brands from raw data to **Strategic Maturity**. 

### Strategic Outcomes:
* Moving away from metrics such as ROAS + Traffic to LTV and Bridge (2nd order) efficiency.
* Identifying when the exact no of days when a customer becomes profitable 
* Understanding the conversion rate from 1st purchase to 2nd purchase
* Identifying one time customers v loyal customers
* Using historic cohort data to predict next years revenue 

## THE FRAMEWORK
The repository is organized into four distinct modules:

| Module | Scope | Description |
| :--- | :--- | :--- |
| **1.0** | **Validation** | Data integrity guardrails ensuring financial accuracy and single-source-of-truth alignment. |
| **2.0** | **Segmentation** | Customer purchase frequency, profit-bucket stratification, and baseline channel health. |
| **3.0** | **Migration & LTV** | Matured milestone tracking (30, 60, 90, 180, 365 days) and year-over-year cohort behavior. |
| **4.0** | **CAC & Payback** | Blended CAC payback windows, marketing efficiency metrics, and capital allocation modeling. |
| **5.0** | **Behavioral Audit** | Order 1 to 2 conversion bridges, repurchase time-lag intervals, and late-money yield auditing. |

## LIGHTWEIGHT SANDBOX: GOOGLE SHEETS LTV TEMPLATE

Before deploying a full cloud infrastructure (BigQuery + Looker Studio), teams can validate their core unit economics, cohort behavior, and bridge metrics locally using the included **Google Sheets LTV Starter Model**. 

This spreadsheet mirrors the logic executed in BigQuery, making it an ideal prototyping sandbox for smaller datasets or pre-pipeline data modelling.

https://docs.google.com/spreadsheets/d/1piQJXYqLxgbeNgUCNmA902QwG2YnptvYeJZIjDwZKyk/edit?usp=sharing

#### Google Sheet 3. `KPI_Dashboard` (Executive Summary)
Surfaces core health KPIs generated from the raw transaction data:
* **Total Acquired Cohort & Bridge Crossings:** Measures conversion performance from 1st to 2nd purchase.
* **Bridge Conversion Rate %:** Evaluates product-market fit and repeat adoption strength.
* **Purchase Velocity & Time-Lag:** Tracks average days required to convert buyers to their second order.
* **Initial vs. Repeat AOV:** Compares upfront basket scale against secondary order value.

### When to Transition from Google Sheets to Google BigQuery

| Evaluation Metric | Google Sheets Starter | BigQuery + Looker Studio |
| :--- | :--- | :--- |
| **Data Volume** | Best for < 25,000 transaction rows | Scalable to millions/billions of records |
| **Execution Speed** | Real-time formula recalculation | Distributed cloud engine (SQL) |
| **Automation** | Manual CSV copy-paste | Automated scheduled pipeline ingestion |
| **Use Case** | Prototyping & quick health checks | Production auditing & executive reporting |


## DATA FOUNDATIONS 
The most critical step is collecting the right data and getting the right data structure. 

### Example: Order-Level Profit Waterfall Analysis

To show how dynamic costs are subtracted upstream before reaching the reporting view, here is a financial waterfall breakdown for a sample order selling a **Smart Jacket** for **£90.00**:

| Cost Component | Type | Amount (£) | Running Balance (£) | Calculation / Sourcing |
| :--- | :--- | :--- | :--- | :--- |
| **Gross Retail Price (AOV)** | Revenue | **+£90.00** | **£90.00** | Initial Cart Value |
| **Discounts & Promo Codes** | Deduction | **-£9.00** | **£81.00** | 10% Welcome Discount code applied at checkout |
| **Net Order Revenue** | Subtotal | **£81.00** | **£81.00** | `Gross Revenue - Discounts` (Top-line revenue exposed to view) |
| **Product COGS** | Operational Cost | **-£25.00** | **£56.00** | Direct unit manufacturing & freight landed cost per SKU |
| **Payment Gateway Fee** | Transaction Fee | **-£1.82** | **£54.18** | Stripe/Shopify Pay fee (~2.0% + £0.20 per transaction) |
| **Pick, Pack & Shipping** | Fulfillment | **-£6.50** | **£47.68** | Warehouse picking labor + outbound courier delivery fee |
| **Packaging & Returns Provision** | Variable Ops | **-£2.50** | **£45.18** | Branded box, unboxing materials, and blended return rate allowance |
| **Order Net Profit** | **Final Margin** | **£45.18** | **£45.18** | **Net Contribution Margin** (Used in reporting view) |

### Data Lineage & Upstream Profit Modeling

While `view_order_grain_master` exposes a clean `profit` column (e.g., the £45.18 calculated above) for LTV cohort modeling, calculating order-level net margin is highly business-specific and computed upstream

#### **Upstream Staging Pipeline**
1. **Raw Order & Line Item Extraction:** Normalizes raw cart transactions, line items, and fulfillment events.
2. **Dynamic COGS & Fulfillment Attribution:** Joins SKU-level product costs, picking/packing expenses, and variable shipping fees.
3. **Gateway & Discount Reconciliation:** Subtracts payment processor fees (e.g., Stripe/Shopify Pay), merchant fees, and distributed discount codes.
4. **Final Order-Level Grain:** Aggregates order-level gross revenue and net profit before handing off to `view_order_grain_master`.

> **Note:** The cohort models, LTV trajectories, and CAC payback calculations in this repository assume `profit` represents true order contribution margin ($\text{Net Sales Revenue} - \text{COGS} - \text{Variable Operational Costs}$).

### Data Requirements

The data requirements to build the **`view_raw_transactional_customer_ordertable`**

* customer_id
* order_date
* revenue
* profit

Key watchouts: 

* The order_date is always YYYY-MM-DD i.e., 2026-06-13
* Revenue and Profit values are not mixed up i.e., profit is not greater than revenue
* Ensure both are numeric not text
* Consistency of customer_id its always lowercase or uppercase and not mixed
* Ensure there’s no trailing space
* No duplicate records

* **Storage Agnostic:** Whether plugged directly into BigQuery or a Google Sheet, missing these checks causes broken joins, silent aggregation errors, or distorted LTV curves.
* **Format Enforcement:** Google Sheets is particularly prone to text-formatted numbers and trailing spaces in string IDs, whereas BigQuery enforces data types strictly at the schema level.

This Google Doc provides an example of the data required, how it should be structured. 

https://docs.google.com/spreadsheets/d/1BIXYFb17sxFHbq_a42uQxtVTX84_Hj-Ee0zOm_t2ArA/edit?usp=sharing

## THE BUSINESS QUESTIONS THAT CAN BE ASKED 

| Business Area | Strategic Question Asked | Visual Module Reference | Dashboard Page Location |
| :--- | :--- | :--- | :--- |
| **Customer Maturity** | What is the cumulative LTV value of an acquired customer at Months 1, 3, 6, and 12? | **Module 3.1 & 3.3** | **Page 2** (Cohort LTV Trajectory & Retention Dynamics) |
| **Payback Velocity** | How long does it take for cumulative LTV profit to exceed blended CAC? | **Module 4.2** | **Page 4** (CAC PayBack & Strategic Outlook) |
| **Predictive Forecasting** | Based on historic retention yields and LTV trajectories, what is the expected revenue growth? | **Module 4.1.0 & 4.2** | **Page 4** (CAC PayBack & Strategic Outlook) |
| **The "Bridge" Efficiency** | What percentage of customers convert from 1st to 2nd purchase, and what is the margin impact? | **Module 4.1.3 & 4.1.5** | **Page 5** (Behavioral Bridges & Audit) |
| **Profit Concentration** | How is revenue and profit distributed across customer value tiers? | **Module 2.5** | **Page 1** (Executive Summary & Acquisition Health) |
| **Retention Economics** | What percentage of annual profit is driven by repeat purchases from legacy cohorts vs new acquisitions? | **Module 4.1.0** | **Page 4** (CAC PayBack & Strategic Outlook) |
| **Marketing Timing** | What is the average time-lag and median re-order interval between consecutive purchases? | **Module 3.4 & 4.1.2** | **Page 3** (Profit Contribution) & **Page 5** (Behavioral Bridges) |

## THE MODULES

### **1.0 Data Integrity & Validation**
* **1.1 Nulls and Blanks:** Detects orphaned orders missing customer identifiers.
* **1.2 Duplicate Transaction:** Identifies duplicates where a transaction occurs twice.
* **1.3 Date Range Validation:** Ensures the dataset fits within expected logical time bounds.
* **1.4 Validating Dates Day0:** Flags future orders and identifies date format flips (DD/MM vs MM/DD).
* **1.5 Financial Validation:** Uses `SAFE_DIVIDE` to prevent system crashes on zero-revenue orders.

### **2.0 Performance & Segmentation**
* **2.1 Cohort Size:** Tracks the volume of customers acquired in specific time periods.
* **2.2 Annual Active Customer:** Measures brand health by identifying active vs. lapsed users.
* **2.3 Frequency Bucket Analysis:** Groups customers by purchase frequency (1x, 2x, 3x+).
* **2.4 Monthly Bucket Analysis:** Tracks how purchase frequency shifts month-over-month.
* **2.5 Profit Bucket Analysis:** Segments the database by actual margin contributed per customer.

### **3.0 LTV & Cohort Maturation**
* **3.1 Cohort LTV Matrix:** LTV trajectory and distribution across historical cohorts.
* **3.2 Cohort Retention Matrix:** Tracks retention curves and cohort repeat purchase rates.
* **3.3 Yearly Benchmark Performance:** Compares cohort quality and LTV side-by-side (e.g., 2023 vs. 2024 vs. 2025).
* **3.4 Days Between Orders:** Measures repurchase latency to optimize marketing campaign timing.
* **3.5 Revenue Profit Contribution:** Evaluates top-line and net margin yields per cohort.

### **4.0 CAC & Payback Milestones**
* **4.1.0 Migration Retention Profitability:** Measures total profit gained as customers move from Order 1 to Order 2+.
* **4.1.6 Monthly Yearly Performance:** High-level executive view for historical trend analysis and long-term forecasting.
* **4.2 CAC Payback Profit Milestone:** Tracks cumulative net profit against CAC to determine exact payback days (30, 60, 90, 180, 365).

### **5.0 Behavioral Bridges & Core Views**
* **4.1.1 Late Money Calendar Yield:** Measures repeat revenue generated by legacy cohorts after their acquisition year.
* **4.1.2 Time Lag First-Last Purchase:** Analyzes customer repurchase windows grouped into operational time buckets.
* **4.1.3 First-to-Second Bridge Financials:** Compares AOV and profit per order from Order 1 to Order 2 and Order 3+.
* **4.1.3b Monthly Bridge Financials:** Tracks month-over-month conversion volume from initial acquisition to first repeat.
* **4.1.4 Daily Weekly Monthly Performance:** Granular macro time-series aggregations for tactical operational tracking.
* **4.1.5 Granular First-Second Order Audit:** Cohort-level audit table detailing acquisition volume, repeat conversions, and net yield.
* **5.1 View Order Grain Master:** Single source of truth view standardizing transaction, revenue, and profit grain.
* **5.2 View Cohort Lifecycle Matrix:** Core reporting view powering cohort and LTV visualizations.
* **5.3 View Data Quality Audit:** Automated view monitoring table integrity and data hygiene.

### Data Pipeline Layers

There are 2 data pipelines that drives the SQL views: 

* **`view_raw_transactional_customer_ordertable`**
  * **Consolidates:** `1.1`, `1.2`, `1.3`, `1.4`, `1.5`, `2.1`, `2.2`, `2.3`, `2.4`, `2.5`, `5.1`, `5.3`, 

* raw transactional order table is core to **`view_order_grain_master`**
  * **Consolidates:** `3.1`, `3.2`, `3.3`, `3.4`, `3.5`, `4.1.0`, `4.1.1`, `4.1.2`, `4.1.3`, `4.1.3b`, `4.1.4`, `4.1.5`, `4.1.6`, `4.2`, `5.2`, `5.3`,
 
**`view_raw_transactional_customer_ordertable`** feeds **`view_order_grain_master`**

## TESTING
To get started I have shared two testing datasets. 

Red Dataset - dataset with many errors

https://docs.google.com/spreadsheets/d/1og04NJxjfeBYWcuTw5WFuoWs5siGREiIdg6Oqa5wMgw/edit?usp=sharing

Run module 1.0 - Data Integrity & Validation it should throw out multiple errors meaning till the data gets fixed it cannot proceed any further 

Green Dataset - the pefect dataset, with clean, structured data 

https://docs.google.com/spreadsheets/d/1aY6ut88A6mO_ZCtAPMZAAxOZxihWw9qk2FjWtaS6SUM/edit?usp=sharing

Run all modules. 4.1.3 should be an interesting one to understand 

## EVOLVING THE DATA REQIREMENTS TO DO GEO + PRODUCT LTV 

To better understand product LTV the data requirements are: 

* customer_id
* order_date
* product_id (sku_id)
* product_name 
* revenue
* profit

To better understand geo LTV the data requirements are: 

* customer_id
* order_date
* city_id/postcode
* city_name 
* revenue
* profit

The master view would comibne customer + product + geo 

* customer_id
* order_date
* product_id(sku_id)
* product_name
* city_id/postcode
* city_name
* revenue
* profit

## LTV STRATEGIC CALCULATOR (SCENARIO PLANNER)
A calculator designed to translate raw customer data into a high-level growth insights. It isolates the "First Order" cost of acquisition from the "Bridge" (2nd order) to lifetime profitability.

https://docs.google.com/spreadsheets/d/1q2Ah-IfcfJ73Ik6Fc-FeK-5Kp9aRrTUNTmpf2V__aiw/edit?usp=sharing

The calculator is dynamic with the goal to better undersrand Profit Per User. It automatically scales based on AOV and Bridge targets, showing exactly how much additional CAC can be afforded as back-end efficiency improves.

## Key Metrics (Section 2)
These metrics represent the **Net Profitability** of your acquisition engine.

| Metric | Unit | Definition |
| :--- | :--- | :--- |
| **Day 0 Net Cash Flow** | £ Total | The total profit or loss for the cohort after subtracting Marketing Costs (CAC) from the initial Gross Profit. |
| **Month 12 Cohort Profit** | £ Total | The estimated total profit from this group after 1 year, including repeat "Bridge" revenue. |
| **Max Allowable CAC** | £ / User | The break-even ceiling; the maximum you can spend to acquire a user while breaking even over 12 months. |
| **Current Profit Gap** | £ / User | The "Marketing Treadmill"—the net profit or loss realized on the very first transaction per user. |

## Growth Levers (Section 4)
This section quantifies the financial impact of specific strategic interventions per user.

| Metric | Unit | Definition |
| :--- | :--- | :--- |
| **Baseline LTV** | £ / User | The current total projected profit value of a single customer over 12 months. |
| **Improve Bridge (+10%)** | £ / User | The incremental profit gained per retained customer by improving the retention bridge efficiency by 10%. |
| **Increase Order 2 AOV** | £ / User | The profit added to every user in the cohort by increasing repeat spend by £10. |
| **Profit Per User** | £ / User | Profit Lift (Orders 1-2). The additional profit realized per user by optimizing the path from the first to the second purchase. |
| **Profit Swing** | % / User | **Turnaround Magnitude.** Measures the total value created by the strategy (Moat + Gap recovery) relative to the initial loss.|

## Calculating Unit Economic Lift
When improving both Retention and AOV, it gains a "compounding bonus", newly retained customers are also spending at the higher AOV.

**The Formula:**

Total Lift = (New AOV × Margin × New Bridge Rate) — (Old AOV × Margin × Old Bridge Rate)

### How to Use Google Sheet - LTV Strategic Calculator

1. **Input Data:** Input into the blue cells: Monthly Users, CAC, AOV, and Margins (Section 1 and 3).
2. **Review the Gap:** Check the **Current Profit Gap**. If it is negative, you are paying a "fee" to acquire customers and relying entirely on the "2nd purchase" to reach profitability.
3. **Simulate Growth:** Profit Per User and Max Allowable CAC provide the **Strategic Insights** to outscale competitors.

## GOOGLE CLOUD - BIG QUERY DATA WAREHOUSE DEPLOYMENT (+ DATA STUDIO)

The standalone analytical models (`1.x` through `4.x`) are compiled into **3 Master BigQuery Views** under section `5.x`. This setup serves as the presentation layer for Data Studio, supporting Executive Mobile, Executive Desktop, and Operational Deep-Dive dashboards.

### Production SQL Architecture (`5.x`)

* **`5.1_view_order_grain_master.sql`**
  * **Dataset Target:** `enter.tablename.customersalesdata.view_order_grain_master`
  * **Consolidates:** `1.2`, `2.2`, `3.4`, `4.1.3`, `4.1.5`, and `4.2`
  * **Purpose:** Establishes an order-level granularity dataset. Computes chronological purchase sequences (`order_sequence`), days to second order (`days_to_next_order`), inter-purchase time gaps (`days_between_orders`), and cumulative realized customer profit. Powers both Executive Scorecard summaries and granular dashboard filters.

* **`5.2_view_cohort_lifecycle_matrix.sql`**
  * **Dataset Target:** `enter.tablename.customersalesdata.view_cohort_lifecycle_matrix`
  * **Consolidates:** `2.1`, `2.3`, `2.4`, `2.5`, `3.1`, `3.2`, `4.1.0`, `4.1.1`, `4.1.2`, and `4.3`
  * **Purpose:** Pre-aggregates cumulative cohort dynamics across standard elapsed maturation buckets (`Day 000` through `Day 365+`). Powers Looker Studio cohort heatmaps, cumulative LTV expansion curves, and CAC payback tracking without client-side lag.

* **`5.3_view_data_quality_audit.sql`**
  * **Dataset Target:** `enter.tablename.customersalesdata.view_data_quality_audit`
  * **Consolidates:** `1.1`, `1.3`, `1.4`, and `1.5`
  * **Purpose:** Serves as an automated data validation monitor. Scans raw transactional ingest for null keys, future-dated records, or revenue/profit discrepancies, exposing these metrics to a hidden Admin Data Health page.
 
## DATA STUDIO DASHBOARDS OVERVIEW

### Dashboard Suite Overview

* **1. Data Engineering Audit Dashboard:** https://datastudio.google.com/reporting/8d5e7d80-2275-41d8-accf-d9404d036c9c - **A dedicated, single-page pipeline health monitor designed to catch data quality issues at ingestion—tracking missing customer IDs, date format errors, financial anomalies, and revenue/profit leakage gaps before data reaches reporting layer**
   
* **2. The Master (Pages 1–6) Dashboard:** https://datastudio.google.com/reporting/ea37df0b-5b9b-46d7-95a7-f9b4a99201b0 - **The comprehensive Single Source of Truth (SSOT) reporting suite covering the full customer analytics lifecycle from executive summaries down to cohort LTV trajectories, payback velocity, conversion time-lags, and monthly financial ledgers**
  
* **3. The Executive One-Pager (Desktop Page 1) Dashboard:** https://datastudio.google.com/reporting/ddbf1183-e24c-4d15-95c5-3d8e8ba31092 - **A streamlined desktop dashboard giving C-suite stakeholders an immediate view of top-line scale, unit economics, and core performance trends**

* **4. The One-Pager (Mobile View) Dashboard:** https://datastudio.google.com/reporting/0cbf4c5c-062f-4041-9589-e46cf7585838 - **A vertically stacked, mobile-optimized view featuring core scorecards and condensed conversion matrices for fast executive checks** 

### Data Studio Calculated Fields Reference

| Page # | Page Name | Looker Studio Calculated Fields & Logic |
| :--- | :--- | :--- |
| **Page 1** | **Executive Summary & Cohort Acquisition Health** | • **`Total Cohort Customers`**: `COUNT_DISTINCT(Customer ID)`<br>• **`Total Revenue`**: `SUM(Gross Revenue)`<br>• **`Total Profit`**: `SUM(Net Profit)`<br>• **`Purchase Frequency Bucket`**: `CASE WHEN Orders = 1 THEN '1 Order' WHEN Orders = 2 THEN '2 Orders' WHEN Orders <= 5 THEN '3-5 Orders' ELSE '6+ Orders' END`|
| **Page 2** | **Cohort LTV Trajectory & Retention Dynamics** | • **`Month X LTV Profit`**: `SUM(Profit in Month X) / COUNT_DISTINCT(Cohort Size)`<br>• **`Retention Rate %`**: `COUNT_DISTINCT(Active Customers in Month X) / COUNT_DISTINCT(Cohort Size)`<br>• **`Avg Cumulative LTV Revenue vs. Profit`**: `SUM(Cumulative Revenue) / COUNT_DISTINCT(Cohort Size)` vs. `SUM(Cumulative Profit) / COUNT_DISTINCT(Cohort Size)`|
| **Page 3** | **Profit Contribution & Repurchase Cadence** | • **`Median Days to Re-order`**: `MEDIAN(DATE_DIFF(Next Order Date, Current Order Date, DAY))`<br>• **`Profit Share %`**: `SUM(Cohort Profit) / SUM(Total Revenue)`<br>• **`Contribution Margin %`**: `SUM(Net Profit) / SUM(Gross Revenue)`<br>• **`Avg Days Between Orders`**: `AVG(DATE_DIFF(Next Order Date, Current Order Date, DAY))` |
| **Page 4** | **CAC PayBack & Strategic Outlook** | • **`90 Days LTV Revenue`**: `SUM(Revenue at Day 90) / COUNT_DISTINCT(Acquired Customers)`<br>• **`12 Months LTV Revenue`**: `SUM(Revenue at Day 365) / COUNT_DISTINCT(Acquired Customers)`<br>• **`CAC Payback Point`**: `IF(Cumulative Profit LTV >= Blended CAC, 1, 0)`<br>• **`Year 1 Migration Yield`**: `SUM(Subsequent Years Repeat Profit) / SUM(Total Profit)`|
| **Page 5** | **Behavioral Bridges & Audit** | • **`Order 1 to 2 CR`**: `COUNT_DISTINCT(Customers >= 2 Orders) / COUNT_DISTINCT(Acquired Customers)`<br>• **`Avg Days to 2nd Purchase`**: `AVG(DATE_DIFF(Order 2 Date, Order 1 Date, DAY))`<br>• **`Late Money Contribution`**: `SUM(Revenue After Day 90) / SUM(Lifetime Revenue)`<br>• **`AOV by Order Sequence`**: `SUM(Revenue) / COUNT(Orders)` grouped by Order Number (`Order 1`, `Order 2`, `Order 3+`)|
