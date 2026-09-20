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
## Date Table Relationship

The DateTable was connected to the Transactions table using:

`DateTable[Date] → Transactions[transaction_date]`

The relationship is:

One-to-many (1:*)

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

## 6. Date Table

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

Product Table Relationship

The ProductTable was connected to the Transactions table using:

`ProductTable[Product_id] → Transactions[product_id]`

The relationship is:

One-to-many (1:*)

`ProductTable` = One side
`Transactions` = Many side
Cross-filter direction = Single

This relationship allows product attributes such as category, product type, and product details to filter transaction-level measures.

The ProductTable was used throughout the dashboard for:

- Revenue by Product Category
- Top 5 Products by Revenue
- Product-level filtering
- Product contribution analysis
