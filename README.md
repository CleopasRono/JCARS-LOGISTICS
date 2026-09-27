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
