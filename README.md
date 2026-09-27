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
| **10.** Vehicle Year contained implausible values, including years below 1990, above 2026, and text such as "twenty twenty". |

