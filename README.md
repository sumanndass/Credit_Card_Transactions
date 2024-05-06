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
- But we will fetch 'address' data from a excel document because we want to fetch data from different location.
- We need to load the data from 'Credit_Card_Transactions' database and from excel document file to 'CC_Transactions_Stage' database using ETL operation.
- Then we will load the data to 'CC_Transactions_DW' data warehouse database.


### Objective


### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - Cube, Pre aggregating values
- SSRS -


### SSIS for Stage Database (SQL Server Integration Service)
- **create a stage database named 'CC_Transactions_Stage' (where ETL will happen)**
  ```sql
  create database pizzeria_stage
  ```

- **use the 'CC_Transactions_Stage' database**
  ```sql
  use pizzeria_stage
  ```

- **Create SSIS Project**
  - Create SSIS project using 'Integration Service Project' in DevEnv

- **Create SSIS Package for Stage Server**
  - open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
  - we have 5 tables in database. So, we need to create 5 packages named '1_city', '2_address', '3_customer', '4_merchant', '5_transactions'.
    <br> **Remember**
    <br> Data Flow - ETL Activities
    <br> Control Flow - Non-ETL Activities

- **Data Loading to 'CC_Transactions_Stage' Database from 'Credit_Card_Transactions' Database**
  - double click on '1_city' SSIS Packages
  - drag 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'OLE DB Source'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'Credit_Card_Transaction' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose '[dbo].[city]' from 'Name of the table or the view' -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from source to destination
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> again select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'CC_Transactions_Stage' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'New' in 'Name of the table or the view'
    - change table name to 'stage_city' and change data type if needed - Ok
    - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
  - change names of connections in 'Connection Managers' for better understanding like for source connection named 'source.Credit_Card_Transaction', for destination connection named 'dest.CC_Transactions_Stage' -> right click on it and 'Convert to Package Connection' for rest of the project
  - stage table always needs fresh data
  - so, drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'dest.CC_Transactions_Stage' in 'Connection'
    - add SQL truncate command (`truncate table stage_city`) to delete all old data from stage whenever new data comes -> Ok
    - connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
    - 'Start' the project
    - do the same for '2_address', '3_customer', '4_merchant', '5_transactions' packages

- **Data Loading to 'CC_Transactions_Stage' Database from 'Excel' Document**
  - double click on '2_address' SSIS Packages
  - drag 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'Excel Source'
    - double click on it
    - in 'connection Manager' select 'New'
    - find the 'excel' document path and choose the same and select proper 'Excel version' -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose 'address$' from 'Name of the Excel sheet' -> Ok
  - drag 'Data Conversion' (because, we need to convert Unicode data to ANSI data)
    - connect 'blue pipe' from 'Excel Source' to 'Data Conversion'
    - double click on it
    - select the 'Available Input Columns'
    - change the 'Data Type' and 'Length' -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from 'Data Conversion' to 'OLE DB Destination'
    - double click on it
    - in 'connection Manager' select 'sourceDB Credit_Card_Transaction' (which was saved earlier) in 'OLE DB Connection manager'
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'New' in 'Name of the table or the view'
    - change table name to 'address_stage' and change data type if needed -> Ok
    - now click on 'Mappings' choose proper 'Input Column' and 'Destination Column' -> Ok
  - change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  - stage table always needs fresh data
  - so, drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'CC_Transactions_Stage' in 'Connection'
    - add SQL truncate command (`truncate table address_stage`) to delete all old data from stage whenever new data comes -> Ok
    - connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
    - 'Start' the project

- **SSIS Logging (To know about the status of the successful loadings)**
  - create a SSIS logging table named 'ssis_log' in stage server where SSIS status will store
    ```sql
    create table ssis_log
    (
      	id			int		primary key identity(1, 1),
      	pkg_name		varchar(100)	not null,
      	pkg_exec_time		datetime	not null,
      	row_cnt			int		not null,
      	pkg_exec_status		varchar(100)	not null
    )
    ```
  - now we will store SSIS loading status in newly created table
    - open '1_city' package and drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'CC_Transactions_Stage' in 'Connection'
    - add 'SQLStatement' command (`insert into ssis_log values(?, getdate(), ?, 'Success...')`) -> Ok
    - in 'Parameter Mapping' click on 'Add' and choose 'System::PackageName' in 'Variable Name' for first '?' means 0th position
    - choose 'Varchar' as 'Data type', choose '0' in 'Parameter Name' for the 0th position '?', choose '-1' for 'Parameter Size' -> Ok
    - now click on 'Variables' and select 'Add variable' -> choose 'Name' like: 'row_cnt'
    - now in 'Data Flow', drag 'Row Count'
    - connect 'blue pipe' from 'OLE DB Source' to 'Row Count'
    - double click on 'Row Count' and select the 'User::row_cnt' variable
    - connect 'blue pipe' from 'Row Count' to 'OLE DB Destination'
    - now in 'Control Flow', double click on newly created 'Execute SQL Task'
    - in 'Parameter Mapping' click on 'Add' and choose 'User::row_cnt' in 'Variable Name' for second '?' means 1st position
    - choose 'Large_integer' as 'Data type', choose '1' in 'Parameter Name' for the 1st position '?', choose '-1' for 'Parameter Size' -> Ok
    - connect 'green pipe' from 'Data Flow Task' to new 'Execute SQL Task'
    - 'Start' the project
    - to check loggings run `select * from ssis_log`
    - do the same for '2_address', '3_customer', '4_merchant', '5_transactions' packages

- **SSIS Failure Logging (To know about the failure happened in loading)**
  - double click on desired package -> go to 'Extension' -> 'SSIS' -> 'Logging'
  - choose all 'Containers:' from left side -> 'Add' 'SSIS log provider for SQL Server', tick the same and choose the destination server 'destDB.pizzeria_stage' in 'Configuration' -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in system tables in the selected database by using command `sql select * from sysssislog`
  - again, choose all 'Containers:' -> 'Add' 'SSIS log provider for Windows Event Log', tick the same -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in 'Windows Event Viewer' -> 'Windows Logs' -> 'Application'
  - Set this task for other packages also


### SSIS for Data Warehouse Database (SQL Server Integration Service)
- **Create Data Warehouse Database**

- **Create Dimension and Fact table in DWH**

- **Create Reference Tables/Views in DWH**

- **Create SSIS Package for DWH Server**

- **Data Loading to 'bank_dw' Database from 'bank_stage' Database**

- 
