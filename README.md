# JCARS-LOGISTICS
Power BI data analytics project transforming a raw Kenyan vehicle dealership dataset into a validated star-schema model, interactive dashboards, and actionable business insights.
## Project Objective
JCars Logistics imports, sells, and delivers vehicles to customers across Kenya. This project transforms a raw and uncleaned vehicle sales dataset into a reliable, interactive Power BI Business Intelligence solution designed to support management decision-making.
The solution provides insights into the following; sales performance, revenue and profitability, branch and geographical performances, vehicle and sales representatives performances, customer value, logistics and delivery efficiency, returns and payment activities and data quality exceptions requiring further investigations.
This projects covers the complete analytics workflow from data cleaning and validation to data modeling, DAX development, visualization, and business insight generation.
## Dataset and Data Grain
The source data is a single flat CSV file, JCARS DATA.csv, containing 32 raw columns from JCars Logistics' operational records
Data Grain

One row represents one vehicle sales transaction (order).
The original dataset combines several business entities into one flat table, including: Customer information, vehicle details, branch information, sales representative information, sales transaction details, payment and delivery information and finally on logistics information. One of the main objectives of the project was to transform this flat structure into a properly designed star schema for power Bi analysis.
## Data Quality Auddit
Before building the data model, i conducted a detailed audit of the source data. The audit revealed several issues that could have affected the accuracy of the final analysis.
### Data Cleansing Log
| Issue Identified | How It Was Handled |
| :--- | :--- |
| **1.** Inconsistent category spellings such as "totoya", "toyta" → Toyota and "cental" → Central across fields such as Make, Model, Region, County, Branch, Payment Status, and Delivery Status. | Created explicit mapping tables in Power Query to standardize affected categorical fields. |
| **2.** Mixed currency formats including KSh, KES, USD, $, and a corrupted currency symbol. | Standardized monetary values using a reusable Power Query parsing function. |
| **3.** Missing values across price, cost, date, and demographic fields. | Applied documented business defaults and added a Data Quality Flag rather than silently imputing values. |
| **4.** Dates stored in multiple formats, including Excel serial numbers, en-GB, en-US, "N/A", and "#DATE!". | Created a reusable fnDate function to parse recognized formats and return null for invalid values. |
| **5.** Delivery dates occurring before order dates on 16 transactions. | Set the invalid delivery dates to null and flagged them instead of inventing replacement dates. |
| **6.** One vehicle recorded with a selling price of KSh 122,420,000 against a cost of KSh 9,382,000. | Identified as a probable data-entry error and corrected to KSh 12,242,000. The correction was documented and traceable. |
| **7.** Order ID was not reliably unique. 19 rows shared "Order ID Missing" and 3 genuine ID collisions were identified. | Defined total orders using COUNTROWS(FactSales) because the dataset grain is one row per transaction. |
| **8.** Revenue Recorded disagreed with calculated Revenue across a significant number of transactions, sometimes by more than 90×. | Investigated the discrepancy and traced much of it to currency inconsistencies. Calculated Revenue was retained as the reporting source of truth. |
| **9.** Numeric fields contained free-text values such as "fifteen", "ten percent", "one", and "two cars". | Created custom parsing functions such as fnDisc and fnUnits to convert recognized values into numbers. |
| **10.** Vehicle Year contained implausible values, including years below 1990, above 2026, and text such as "twenty twenty". | Parsed values where a clear interpretation existed and treated implausible values as invalid|
This audit reinforced an important principle throughout the project: data cleaning is not enough; the cleaned data must also be validated against business logic and independent calculations.
## Currency Standardization
One of the most significant issues discovered during the audit was inconsistent currency representation. The monetary columns contained a mixture of: Unmarked values, KES, KSH, USD, $ and a corrupted currency symbol caused by CSV text encoding issue. Unmarked monetary values were treated as Kenyan Shillings (KES) based on the assumptions provided in the assessment brief.
### Approach
A reusable Power Query function, `fnNum`, was developed to:
* **Detect currency symbols:** Identify `USD` and `$` markers before removing non-numeric characters.
* **Preserve data integrity:** Keep valid numeric characters intact during processing.
* **Support scale suffixes:** Handle values expressed in millions by parsing the `M` suffix.
* **Standardize data types:** Convert cleaned text values into strict numeric data types.
* **Normalize currency:** Standardize and convert USD-denominated values into KES.
* **Handle exceptions:** Return `null` safely when a value cannot be reliably interpreted.

### Assumptions and Business Rules

The following assumptions were applied consistently throughout the project:

* **Default Currency:** Monetary values without an explicit currency marker are assumed to be in KES.
* **Customer Identification:** A customer is defined as a unique combination of Name, Type, Age, Region, County, and City, because the source dataset does not contain a unique customer identifier.
* **Demographic Defaults:** Missing Customer Age defaults to **35**.
* **Vehicle Data Defaults:** Missing Vehicle Year defaults to **2022**.
* **Transaction Defaults:** Missing Units Sold defaults to **1**.
* **Financial Overheads Defaults:** Missing Discount, Review Count, Delivery Fee, and Logistics Cost default to **0**.
* **Feedback Defaults:** Missing Customer Rating defaults to **3**, representing a neutral midpoint.
* **Pricing & Cost Defaults:** Missing Unit Selling Price and Unit Cost default to **0** and are flagged as estimated values.
* **Margin Exclusions:** Records with zero selling price are excluded from per-unit margin analysis because KSh 0 does not represent a meaningful business outcome.
* **Geographic Grain:** Region, County, and Branch are treated as separate fields. Region and County represent customer geography because the source data does not directly associate geography with Branch.
* **Order Metric Definition:** Total Orders is calculated using `COUNTROWS(FactSales)` because one row represents one transaction.
* **Financial Calculations:** Gross Profit is defined as `Total Revenue − Total Unit Cost`. Logistics and delivery costs are tracked separately.
## 6. Data Model
Once the data cleaning and validation were complete, I moved on to building the data model. The original dataset was a single flat table containing customer, vehicle, branch, sales representative, and transaction information. To make the data easier to analyze and maintain in Power BI, I transformed it into a **star schema**.

At the center of the model is the **`FactSales`** table, with one row representing each sales transaction. It contains the key transactional information needed for analysis, including revenue, unit cost, units sold, discounts, logistics costs, delivery fees, and the foreign keys connecting each transaction to the relevant dimension tables.

Surrounding the fact table are five dimension tables: **`DimCustomer`** for customer information, **`DimVehicle`** for vehicle make, model, year, and type, **`DimBranch`** for branch details, **`DimSalesRep`** for sales representative information, and **`DimDate`**, a custom calendar table used for time-based analysis.

The relationships were configured as **one-to-many, single-direction relationships**, with the dimension tables filtering the `FactSales` table. This structure keeps the model organized and makes it easier to build reliable DAX measures and interactive reports.

Another challenge was that the source data did not provide reliable unique identifiers for the dimension entities. To solve this, I created **surrogate keys** such as `CustomerID`, `VehicleID`, `BranchID`, and `SalesRepID` using Power Query's Index Column feature.
One important lesson here was the order of operations. I had to **deduplicate the records before generating the index**. If the index is created first, every row receives a unique number, making the rows appear different even when they contain duplicate information. By deduplicating first and then generating the surrogate keys, I was able to create cleaner and more reliable dimension tables.

### Key DAX Measures

The following DAX measures were created to support the analysis:

#### Total Revenue
```dax
Total Revenue = SUM(FactSales[Revenue])
```

#### Total Unit Cost
```dax
Total Unit Cost = 
SUMX(
    FactSales,
    FactSales[Units Sold] * FactSales[Unit Cost]
)
```

#### Gross Profit
```dax
Gross Profit = [Total Revenue] - [Total Unit Cost]
```

#### Gross Profit Margin
```dax
Gross Profit Margin = 
DIVIDE([Gross Profit], [Total Revenue])
```

#### Total Orders
```dax
Total Orders = 
COUNTROWS(FactSales)
```

#### Avg Order Value
```dax
Avg Order Value = 
DIVIDE([Total Revenue], [Total Orders])
```

#### Total Units Sold
```dax
Total Units Sold = 
SUM(FactSales[Units Sold])
```

#### Total Returns
```dax
Total Returns = 
CALCULATE(
    [Total Orders],
    FactSales[Returned] = "Yes"
)
```

#### Return Rate
```dax
Return Rate = 
DIVIDE([Total Returns], [Total Orders])
```

#### Cancelled Revenue
```dax
Cancelled Revenue = 
CALCULATE(
    [Total Revenue],
    FactSales[Delivery Status] = "Cancelled"
)
```

#### Revenue PY
```dax
Revenue PY = 
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR(DimDate[Date])
)
```

#### Revenue YoY %
```dax
Revenue YoY % = 
DIVIDE(
    [Total Revenue] - [Revenue PY],
    [Revenue PY]
)
```

#### Total Logistics Cost
```dax
Total Logistics Cost = 
SUM(FactSales[Logistics Cost])
```

#### Logistics Cost % of Revenue
```dax
Logistics Cost % of Revenue = 
DIVIDE(
    [Total Logistics Cost],
    [Total Revenue]
)
```

#### Customer Revenue Rank
```dax
Customer Revenue Rank = 
RANKX(
    ALL(DimCustomer[Customer Name]),
    [Total Revenue]
)
```

## Executive Dashboard

Once the data model and DAX measures were in place, I moved on to building the **Executive Dashboard**, which was designed to give management a quick overview of how the business was performing. I wanted the page to answer the most important questions at a glance, so I included KPI cards showing **Total Revenue, Gross Profit, Gross Profit Margin, Total Units Sold, Total Orders, and Return Rate**. 

Alongside these KPIs, I added a monthly revenue trend to show how sales were changing over time, as well as comparisons of revenue by branch and vehicle make. To make the dashboard interactive, I included slicers for **date range, branch, and vehicle type**, allowing users to quickly filter the results and explore specific areas of the business. The idea was to keep the first page focused on the bigger picture, giving management an immediate understanding of overall performance before they moved into the more detailed pages of the report.

## Detailed Report Pages

After building the Executive Dashboard, I created three additional pages to allow users to explore the business in more detail. The first page, **Sales, Profitability & Vehicles**, focuses on vehicle performance across Make, Model, and Vehicle Type, while also exploring sales, profitability, and the relationship between discounts and margins. The second page, **Branches, Reps & Customers**, looks at branch and geographic performance, sales representative activity, lead source effectiveness, and the contribution of top customers. The third page, **Logistics, Payments & Exceptions**, focuses on payment and delivery status, logistics cost efficiency, returns, and operational exceptions, with a dedicated view of transactions that may require further management attention.

## Interactivity

I also wanted the report to be more than a collection of static charts, so I added several interactive features to make it easier for users to explore the data. Slicers allow users to filter the report by **date range, branch, vehicle type, fuel type, payment status, and delivery status**, depending on the page. I also added a **drillthrough** feature that allows users to select a customer from the customer analysis page and move directly to the Exceptions page while keeping that customer's filter context. In addition, I created a **custom tooltip** for the Revenue-by-Make visual, which displays a small revenue trend when users hover over a vehicle make, providing additional context without requiring them to leave the current page.

## Analyst-Defined Questions

Beyond the questions provided in the assessment brief, I identified **five additional business questions** that I felt were worth exploring. Three of these were incorporated into the Power BI solution. The purpose was to move beyond simply describing what had happened in the data and instead investigate areas such as **profitability, customer behavior, operational performance, and business exceptions**. This helped me approach the dashboard from a business perspective rather than simply building visuals around the available columns.

## Key Insights

Once the analysis was complete, several findings stood out. The most significant was the **currency inconsistency** in the source data, which caused some transactions to be overstated or understated by approximately **130–150×** before the issue was corrected. I also identified a single vehicle whose selling price had been overstated by **10×**, which appeared to be a data-entry error and would have been difficult to identify through basic inspection alone.

Another important finding was the unreliability of the manually entered `Revenue Recorded` field. It disagreed with the calculated revenue on approximately **40% of transactions**, with some differences exceeding **90×**. Because of this, I used the validated calculated revenue as the source of truth for the analysis rather than relying on the manually recorded figure.

The overall **return rate was approximately 32%**, which highlighted an area requiring further investigation across branches, vehicle years, sales representatives, and other relevant dimensions. I also identified discount-band analysis as an additional area for exploration, particularly in understanding how discount levels may relate to profitability. Finally, the overall **gross profit margin was approximately 15%**, providing a useful baseline for monitoring profitability over time and understanding how changes in pricing and discounting may affect margins.

Together, these findings showed why the validation stage was such an important part of the project. Some of the most important insights were not immediately visible in the raw data or dashboard; they emerged only after I started questioning the numbers and comparing them against other measures and business logic.

## Management Recommendations

The analysis did more than highlight problems in the data; it also pointed to several areas where JCars Logistics could strengthen its reporting and operational processes.

The first recommendation is to **standardize currency capture at the point of sale**. The dataset contained a mixture of KES, USD, and other currency representations, which created significant differences in calculated revenue. Introducing a mandatory currency field, with KES as the default where appropriate, would reduce ambiguity and make currency conversion much easier to manage consistently.

The second recommendation is to **introduce basic price validation rules during data entry**. The extreme vehicle price outlier showed how a simple entry error can have a major impact on analysis. A validation rule could flag prices that fall significantly outside the typical range for a particular Make or Model, allowing the transaction to be reviewed before it reaches the reporting stage.

The **approximately 32% return rate** also deserves further investigation. Rather than assuming that all returns have the same cause, management could break the figure down by branch, vehicle year, sales representative, vehicle type, and other relevant factors. This would help identify whether the returns are concentrated in particular areas and provide better context before any corrective action is taken.

Another important recommendation is to **use system-calculated revenue as the primary reporting figure**. Since the manually recorded `Revenue Recorded` field disagreed with calculated revenue across a significant number of transactions, relying on the calculated value provides a more consistent basis for reporting, while the original field can still be retained for audit and comparison purposes.

Finally, I would recommend **building automated data-quality checks into the regular reporting process**. Issues such as mixed currencies, impossible dates, duplicate identifiers, missing values, and unusual prices should ideally be detected as part of routine data processing rather than only when someone performs a detailed manual audit. This would make the reporting process more reliable and reduce the risk of incorrect data making its way into management decisions.

### Repository Contents

```text
JCars-Logistics-A-Power-BI-Business-Intelligence-Project/
│
├── Jcar_Project.pbix
├── Jcars_data.csv
├── README.md
│
└── screenshots/
    ├── 01-model-view.png
    ├── 04-executive-dashboard.png
    ├── 05-page2.png
    ├── 06-page3.png
    ├── 07-page4.png
    ├── 08-drillthrough.png
    ├── 09-tooltip.png
    └── 10-dax-measures.png
```

#### Files Description
* **`Jcar_Project.pbix`:** Completed Power BI project file containing the data model, Power Query steps, and interactive dashboards.
* **`Jcars_data.csv`:** Source dataset used for the analysis and data cleansing processes.
* **`README.md`:** Comprehensive project documentation and documentation logs.
* **`screenshots/`:** Folder containing high-resolution report, dashboard, and relationship model screenshots.

## Challenges and Key findings.

This project went beyond standard data cleaning and required a significant amount of **forensic data investigation**. Some of the most important issues, particularly the currency inconsistencies and revenue discrepancies, were not immediately visible when I first inspected the raw dataset. They only became apparent when I started comparing calculated values against independently recorded figures and testing whether the results made sense from a business perspective.

One of the biggest lessons I took from the project is that **a dataset can look clean without actually being correct**. A column can have the right data type, consistent formatting, and no obvious errors while still producing misleading results. This made data validation one of the most important parts of the entire process rather than simply a final step after cleaning.

Throughout the project, I strengthened my practical skills in **Power Query, data cleaning and transformation, Power BI data modeling, star-schema design, DAX, data validation, and interactive dashboard development**. More importantly, I learned how to connect technical data-quality issues to actual business questions and recommendations.

Ultimately, the project showed me that building a useful Power BI solution is not just about creating attractive dashboards. It is about creating a reliable path from **raw data to validated information and, finally, to meaningful business insight**. The dashboard is only as trustworthy as the data and validation process behind it.
