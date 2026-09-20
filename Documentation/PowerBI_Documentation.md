# Power BI Documentation — Coffee Café Sales & Transaction Analysis

## 1. Project Overview

This document provides the technical and analytical documentation for the **Power BI** component of the Coffee Café Sales & Transaction Analysis project.

The objective was to transform raw café transaction data into an interactive business intelligence dashboard that provides insights into:

- Overall sales performance
- Monthly revenue trends
- Product and category performance
- Store-level revenue performance
- Transaction value
- Business areas requiring further investigation

The dashboard was designed with a focus on **clear business storytelling**, rather than displaying every analysis performed during the development process.

---

## 2. Dataset Overview

The analysis uses a coffee café transaction dataset containing **149,116 transaction records** covering the period from **January 1, 2023 to June 30, 2023**.

### Dataset Statistics

| Metric | Value |
|---|---:|
| Total Transactions | 149,116 |
| Total Quantity Sold | 214,470 |
| Total Revenue | $698,812.33 |
| Average Transaction Value | $4.69 |
| Store Locations | 3 |
| Product Categories | 9 |
| Products | 80 |
| Dataset Columns | 11 |
| Analysis Period | January–June 2023 |

### Dataset Columns

| Column | Description |
|---|---|
| `transaction_id` | Unique identifier for each transaction |
| `transaction_date` | Date on which the transaction occurred |
| `transaction_time` | Time at which the transaction occurred |
| `transaction_qty` | Quantity of products purchased |
| `store_id` | Identifier of the store |
| `store_location` | Location of the store |
| `product_id` | Identifier of the product |
| `unit_price` | Selling price per unit |
| `product_category` | Category of the product |
| `product_type` | Type of product |
| `product_detail` | Detailed product name |

---

## 3. Data Quality Checks

Before building the Power BI model, the dataset was checked for common data-quality issues.

### Checks Performed

- Duplicate rows
- Duplicate transaction IDs
- Missing values
- Data types
- Date range
- Numerical ranges
- Number of product categories
- Number of products
- Number of store locations

### Results

| Check | Result |
|---|---:|
| Duplicate Rows | 0 |
| Duplicate Transaction IDs | 0 |
| Null Values | 0 |
| Unique Transaction IDs | 149,116 |
| Date Range | Jan 1, 2023 – Jun 30, 2023 |
| Product Categories | 9 |
| Products | 80 |
| Store Locations | 3 |

The dataset did not require major data-cleaning operations because there were no duplicate records or missing values.

---

## 4. Data Model

The Power BI model uses a simple dimensional structure consisting of:

- `Transactions`
- `DateTable`
- `ProductTable`

### Transactions

The `Transactions` table contains the original transaction-level data.

It acts as the primary **fact table** for the analysis.

### DateTable

A dedicated date table was created to support time-based analysis.

The table contains:

- Date
- Year
- Month Number
- Month Name
- Year-Month

The DateTable was connected to the transaction table using:

`DateTable[Date] → Transactions[transaction_date]`

Relationship:

**One-to-many (1:*)**

### ProductTable

A separate product dimension was created to organize product attributes.

The ProductTable contains:

- Product ID
- Product Category
- Product Details
- Product Type

The ProductTable was connected to the transaction table using:

`ProductTable[Product_id] → Transactions[product_id]`

Relationship:

**One-to-many (1:*)**

### Model Structure

```text
DateTable
    │
    │ 1 : *
    ▼
Transactions
    ▲
    │ 1 : *
ProductTable
```
---

## 5. Date Table

A dedicated `DateTable` was created to support time-based analysis in Power BI.

The table provides the following fields:

- `Date`
- `Year`
- `Month Number`
- `Month Name`
- `Year-Month`

### DAX Used to Create the Date Table

```DAX
DateTable =
ADDCOLUMNS(
    CALENDAR(
        MIN(Transactions[transaction_date]),
        MAX(Transactions[transaction_date])
    ),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Year-Month", FORMAT([Date], "YYYY-MM")
)
```
### Date Table Relationship

The DateTable was connected to the Transactions table using:

`DateTable[Date] → Transactions[transaction_date]`

The relationship is:

#### One-to-many (1:*)

`DateTable` = One side
`Transactions` = Many side
Cross-filter direction = Single

This relationship allows transaction-level measures to be analyzed by date, month, and year.

Month Sorting

The `Month Name` column was sorted by `Month Number`.

This ensures that months appear in chronological order:

`January
February
March
April
May
June`

instead of being displayed alphabetically.

The `Year-Month` column was also created to support chronological month-level filtering and analysis.

---

## 6. Product Table

A separate `ProductTable` was created to organize product-related attributes and support product-level analysis.

The table provides the following fields:

- `Date`
- `Year`
- `Month Number`
- `Month Name`
- `Year-Month`

### DAX Used to Create the Date Table

```DAX
ProductTable =
DISTINCT(
    SELECTCOLUMNS(
        Transactions,
        "Product_id", Transactions[product_id],
        "Product_Category", Transactions[product_category],
        "Product_Details", Transactions[product_detail],
        "Product_Type", Transactions[product_type]
    )
)
```
The resulting ProductTable contains 80 unique products.

### Product Table Relationship

The ProductTable was connected to the Transactions table using:

`ProductTable[Product_id] → Transactions[product_id]`

The relationship is:

#### One-to-many (1:*)

`ProductTable` = One side
`Transactions` = Many side
Cross-filter direction = Single

This relationship allows product attributes such as category, product type, and product details to filter transaction-level measures.

The ProductTable was used throughout the dashboard for:

- Revenue by Product Category
- Top 5 Products by Revenue
- Product-level filtering
- Product contribution analysis

---

## 7. DAX Measures

DAX measures were created to calculate the key business metrics used throughout the Power BI analysis.

The measures were designed to provide reusable calculations that respond dynamically to filters and slicers applied to the dashboard.

### 7.1 Total Product Quantity

```DAX
Total_Product_quantity =
SUM(Transactions[transaction_qty])
```

This measure calculates the total number of product units sold.

#### Result:

#### 214,470 units

---
7.2 Total Transactions
```DAX
Total_transactions =
DISTINCTCOUNT(Transactions[transaction_id])
```
This measure calculates the number of unique transactions.

#### Result:

#### 149,116 transactions

#### Using `DISTINCTCOUNT` ensures that each transaction ID is counted once.
---
7.3 Total Revenue
```DAX
Total_Revenue =
SUMX(
    Transactions,
    Transactions[transaction_qty] * Transactions[unit_price]
)
```
This measure calculates total revenue by multiplying the quantity sold by the unit price for each transaction and then summing the results.

#### Result:
#### $698,812.33
---
7.4 Average Transaction Value
```DAX
Average_Transaction_Value =
DIVIDE(
    [Total_Revenue],
    [Total_transactions]
)
```
This measure calculates the average revenue generated per transaction.

####  Result:
#### $4.69
---
7.5 Revenue Contribution %
```DAX
Revenue Contribution % =
DIVIDE(
    [Total_Revenue],
    CALCULATE(
        [Total_Revenue],
        ALL(ProductTable)
    )
)
```
This measure was created to evaluate the contribution of products or product categories to overall revenue.

#### The `ALL(ProductTable)` function removes the product-level filter context when calculating the overall revenue denominator.
---
7.6 Month-over-Month Revenue Growth
```DAX
MoM Revenue Growth % =
VAR Current_Month = [Total_Revenue]
VAR Previous_Month =
    CALCULATE(
        [Total_Revenue],
        DATEADD(DateTable[Date], -1, MONTH)
    )
RETURN
    DIVIDE(
        Current_Month - Previous_Month,
        Previous_Month
    )
```
This measure calculates the percentage change in revenue compared with the previous month.

It was used during the analytical stage to identify periods of revenue growth and decline.

DAX Summary
Measure	Purpose
`Total_Product_quantity`	Calculates total units sold
`Total_transactions`	Calculates unique transaction count
`Total_Revenue`	Calculates total sales revenue
`Average_Transaction_Value`	Calculates average revenue per transaction
`Revenue Contribution %`	Measures contribution to total revenue
`MoM Revenue Growth %`	Measures month-over-month revenue change

---

# Section 8

## 8. Executive Dashboard

The final Power BI dashboard was designed as an executive-level summary of the café's sales performance.

The dashboard intentionally focuses on the most important business questions rather than displaying every analysis performed during the project.

### Dashboard Structure

The dashboard contains:

- 4 KPI cards
- 4 analytical visuals
- 3 interactive slicers

---

### 8.1 KPI Cards

Four KPI cards were used to provide an immediate overview of business performance.

| KPI | Value |
|---|---:|
| Total Revenue | $698.81K |
| Total Transactions | 149.12K |
| Quantity Sold | 214.47K |
| Average Transaction Value | $4.69 |

These KPIs provide a high-level understanding of the overall sales performance before users explore the detailed visuals.

---

### 8.2 Monthly Revenue Trend

A line chart was used to analyze revenue performance across the six-month period.

**Visual:** Line Chart

**Axis:** Month

**Measure:** Total Revenue

The visual helps identify monthly increases and decreases in revenue.

Key observations include:

- February recorded the lowest monthly revenue.
- Revenue increased after February.
- June recorded the highest monthly revenue.

---

### 8.3 Top 5 Products by Revenue

A column chart was used to identify the five individual products generating the highest revenue.

**Visual:** Clustered Column Chart

**Axis:** Product Details

**Measure:** Total Revenue

A Top N filter was applied to display only the five highest-revenue products.

---

### 8.4 Revenue by Product Category

A horizontal bar chart was used to compare revenue across product categories.

**Visual:** Clustered Bar Chart

**Axis:** Product Category

**Measure:** Total Revenue

This visual makes it easy to compare the relative contribution of each category.

---

### 8.5 Revenue by Store Location

A horizontal bar chart was used to compare revenue generated by the three café locations.

**Visual:** Clustered Bar Chart

**Axis:** Store Location

**Measure:** Total Revenue

This visual allows users to compare store-level revenue performance.

---

### 8.6 Interactive Slicers

Three slicers were added to make the dashboard interactive:

- Category
- Store
- Month

Users can select a specific category, store, or month and observe how the dashboard metrics and visuals change accordingly.

---
## 9. Key Findings

The Power BI analysis produced several important observations from the transaction data.

### 9.1 Overall Business Performance

The café generated approximately **$698.81K in revenue** from **149.12K transactions**.

The total quantity sold was approximately **214.47K units**, with an average transaction value of approximately **$4.69**.

---

### 9.2 Product Category Performance

**Coffee** was the highest revenue-generating product category.

Coffee generated approximately **$270K** in revenue, making it the largest contributor among the nine product categories.

---

### 9.3 Monthly Revenue Performance

**June** recorded the highest monthly revenue at approximately **$166K**.

**February** recorded the lowest monthly revenue at approximately **$76K**.

The data shows a noticeable increase in revenue after February.

However, the transaction dataset alone does not provide enough information to determine the specific reason for the February decline.

---

### 9.4 Store Performance

**Hell's Kitchen** generated the highest revenue among the three store locations, with approximately **$236.5K** in revenue.

The three stores had relatively close revenue totals, but their sales quantity rankings were not identical.

This demonstrates why both revenue and quantity should be considered when evaluating store performance.

---

### 9.5 Product Performance

**Sustainably Grown Organic Lg** was the highest-revenue individual product in the analysis.

The Top 5 Products by Revenue visual provides a focused view of the products making the largest contribution to sales.

---

### 9.6 Revenue vs Quantity

Revenue and quantity do not always produce the same ranking.

A product or store can sell a larger quantity but generate less revenue if its average selling price or product mix is different.

This highlights the importance of analyzing both sales volume and revenue rather than relying on a single metric.

---

## 10. Business Recommendations

The following recommendations are based on the transaction-level analysis.

### 10.1 Monitor High-Performing Categories

The Coffee category is the largest revenue contributor and should continue to be monitored closely.

Further analysis could examine:

- Product-level performance within the category
- Monthly category trends
- Store-level category performance
- Average selling price

---

### 10.2 Investigate February Revenue Performance

February recorded the lowest monthly revenue during the analysis period.

Additional business data should be analyzed to understand the underlying factors.

Potential areas for investigation include:

- Customer traffic
- Promotions and discounts
- Product availability
- Seasonal patterns
- Store operating conditions

The available transaction data alone should not be used to claim a specific cause.

---

### 10.3 Analyze Store Product Mix

Revenue and quantity rankings differ across stores.

A deeper analysis of product mix, pricing, and category performance by store could help explain these differences.

---

### 10.4 Monitor High-Revenue Products

High-revenue products should be monitored regularly to understand their contribution to total revenue.

Further analysis could include:

- Sales volume
- Average selling price
- Store distribution
- Monthly performance
- Category contribution

---

### 10.5 Combine Additional Business Data

The current analysis is based primarily on transaction data.

Additional datasets such as customer traffic, promotions, inventory, costs, and profit would allow a more comprehensive business analysis.

---
## 11. Dashboard Design Principles

The final dashboard was intentionally designed to be concise and easy to interpret.

Instead of displaying every chart created during the analysis process, the final dashboard contains only the visuals that directly support the main business questions.

### Business Questions Addressed

The dashboard answers four primary questions:

1. **How is the business performing?**
2. **How is revenue changing over time?**
3. **Which products and categories generate revenue?**
4. **Which stores generate the most revenue?**

### Design Approach

The dashboard uses:

- Consistent blue visual elements
- Clear visual titles
- KPI cards for headline metrics
- Horizontal bar charts for category and store comparisons
- A line chart for monthly trends
- A Top 5 visual for product analysis
- Dropdown slicers for interactive filtering

The dashboard was kept visually simple so that the main insights can be understood quickly.

---
## 12. Tools Used

The following tools were used to develop the Power BI analysis:

### Power BI

- Power BI Desktop
- DAX
- Data modeling
- Interactive dashboards
- Slicers
- KPI cards
- Time-series analysis
- Top-N analysis

### Microsoft Excel

The original transaction dataset was provided in Excel format and used as the source data for the analysis.

### GitHub

GitHub is being used to document and present the completed analytics project as part of a Data Analytics portfolio.

---
## 13. Limitations

The analysis has several limitations that should be considered when interpreting the results.

### Limited Time Period

The dataset covers only **January 2023 to June 2023**, representing six months of transaction data.

Therefore, the analysis does not provide a full-year view of business performance.

### No Customer-Level Information

The dataset does not contain customer identifiers or customer demographics.

Therefore, customer retention, customer segmentation, and individual customer behavior cannot be analyzed.

### No Promotion or Discount Information

The dataset does not contain detailed promotion or discount information.

Therefore, the effect of promotions on revenue cannot be directly evaluated.

### No Cost or Profit Information

The dataset contains selling prices but does not provide product costs.

Therefore, profitability and profit margins cannot be calculated.

### Limited Causal Analysis

The dataset can identify patterns and relationships in sales performance, but additional business information would be required to determine the causes behind specific changes in revenue.

---
## 14. Project Status

### Power BI

**Dashboard:** ✅ Completed  
**Analysis:** ✅ Completed  
**Documentation:** 🔄 In Progress

### Tableau

**Dashboard:** 🔄 Planned  
**Analysis:** 🔄 Planned  
**Documentation:** 🔄 Planned

### Portfolio

**GitHub Repository:** 🔄 In Progress  
**Presentation:** 🔄 Planned

The Tableau implementation will be developed and documented separately using the same source dataset.

---
