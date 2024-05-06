# Credit_Card_Transactions_MSBI
- This dataset is a simulated credit card transaction dataset that includes both legitimate and fraudulent transactions.
- Using SSIS we will Extract, Transform and Load the data from OLTP server to Stage server and then OLAP/DWH serevr.
- Using SSAS we will store pre-aggregated values (Totals) for multi-dimensional analysis.
- Using SSRS we will produces formatted reports with the tables of data, graph, and reports, ensuring that stakeholders have the information they want as soon as possible.


## Table of Content
- [Overview](#overview)
- [Objective](#objective)
- [Tools Used](#tools-used)
- [SSIS for Stage Database (SQL Server Integration Service)](#ssis-for-stage-database-sql-server-integration-service)
- [SSIS for Data Warehouse Database (SQL Server Integration Service)](#ssis-for-data-warehouse-database-sql-server-integration-service)
- [SSIS Deployment](#ssis-deployment)
- [SSAS Create Cube](#ssas-create-cube)


### Overview
- The dataset chosen for this purpose includes synthetically generated credit card transactions, containing transaction, customer, and merchant details.
- Assume data are storing in various tables in 'Credit_Card_Transactions' database. Tables are 'city', 'address', 'customer', 'merchant', 'transactions'.
- We need to load the data from 'Credit_Card_Transactions' database to 'Credit_Card_Transactions_Stage' database using ETL operation.
- Then we will load the data to 'Credit_Card_Transactions_DW' data warehouse database.


### Objective


### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - Cube, Pre aggregating values
- SSRS -


### SSIS for Stage Database (SQL Server Integration Service)
- **create a stage database named 'pizzeria_stage' (where ETL will happen)**

- **use the 'pizzeria_stage' database**

- **Create SSIS Project**

- **Create SSIS Package for Stage Server**
- 
	<br> **Remember**
  <br> Data Flow - ETL Activities
	<br> Control Flow - Non-ETL Activities

- **Data Loading to 'pizzeria_stage' Database from 'pizzeria' Database**

- **Data Loading to 'pizzeria_stage' Database from 'Excel' Document**

- **SSIS Logging (To know about the status of the successful loadings)**

- **SSIS Failure Logging (To know about the failure happened in loading)**


### SSIS for Data Warehouse Database (SQL Server Integration Service)
- **Create Data Warehouse Database**

- **Create Dimension and Fact table in DWH**

- **Create Reference Tables/Views in DWH**

- **Create SSIS Package for DWH Server**

- **Data Loading to 'bank_dw' Database from 'bank_stage' Database**

- 
