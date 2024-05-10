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
  - One customer can have only one address, while one address can refer to multiple customers who may live in the same address.
  - One address can only have one city, while a city can have multiple addresses.
  - One customer can initiate multiple transactions, while a single transaction must have only one customer.
  - A transaction requires a single merchant, while a merchant can be involved in multiple transactions.
- But we will fetch 'address' data from a excel document because we want to fetch data from different location.
- We need to load the data from 'Credit_Card_Transactions' database and from excel document file to 'CC_Transactions_Stage' database using ETL operation.
- Then we will load the data to 'CC_Transactions_DW' data warehouse database.
  - Deals with historical data
  - Data is organized in multi-dimensional schemas such as stars, snowflake, galaxy/fact constellation, etc
  - The primary use cases for this include business intelligence reporting and business planning


### Objective


### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - Cube, Pre aggregating values
- SSRS -


### SSIS for Stage Database (SQL Server Integration Service)
- **create and use a stage database named 'CC_Transactions_Stage' (where ETL will happen)**
  ```sql
  create database CC_Transactions_Stage
  go
  
  use CC_Transactions_Stage
  go
  ```

- **Create SSIS Project**
  - Create SSIS project using 'Integration Service Project' in DevEnv

- **Create SSIS Package for Stage Server**
  - open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
  - we have 5 tables in database. So, we need to create 5 packages named '1_city', '2_address', '3_customer', '4_merchant', '5_transactions'.
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/8268bec8-3182-4480-9600-9b8f9f961fab)

    <br> **Remember**
    <br> Data Flow - ETL Activities
    <br> Control Flow - Non-ETL Activities
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/4791440a-83ca-4458-8fbe-141654f4a61c)

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
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/e9743fca-5467-460c-b5d8-55747ecdc33b)
  - change names of connections in 'Connection Managers' for better understanding like for source connection named 'sourceDB.Credit_Card_Transaction', for destination connection named 'destDB.CC_Transactions_Stage' -> right click on it and 'Convert to Package Connection' for rest of the project
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/1628b382-4ccd-4a61-8e7f-c6b5f5ae08ec)
  - stage table always needs fresh data
  - so, drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'destDB.CC_Transactions_Stage' in 'Connection'
    - add SQL truncate command (`truncate table stage_city`) to delete all old data from stage whenever new data comes -> Ok
    - connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
    - 'Start' the project
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/8b67dbca-8210-476f-b4d4-fbb9c3676976)
    - do the same for '3_customer', '4_merchant', '5_transactions' packages

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
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/fdc53028-9655-4833-acb2-cd4e148952be)
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
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/8f086e62-2964-4947-a7d2-dd05b90269a3)
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
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/284b39f8-a7d6-47c9-a502-b2a8fb277259)
    - to check loggings run `select * from ssis_log`
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/bed04a16-2dc7-4466-a45e-d075576469c0)
    - do the same for '2_address', '3_customer', '4_merchant', '5_transactions' packages

- **SSIS Failure Logging (To know about the failure happened in loading)**
  - double click on desired package -> go to 'Extension' -> 'SSIS' -> 'Logging'
  - choose all 'Containers:' from left side -> 'Add' 'SSIS log provider for SQL Server', tick the same and choose the destination server 'destDB.CC_Transactions_Stage' in 'Configuration' -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in system tables in the selected database by using command `sql select * from sysssislog`
  - again, choose all 'Containers:' -> 'Add' 'SSIS log provider for Windows Event Log', tick the same -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in 'Windows Event Viewer' -> 'Windows Logs' -> 'Application'
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/841d4b0d-9646-46a0-8187-d333230e468b)
  - Set this task for other packages also


### SSIS for Data Warehouse Database (SQL Server Integration Service)
- **Create Data Warehouse Database**
  ```sql
  create database CC_Transactions_DW
  go

  use CC_Transactions_DW
  go
  ```

- **Create Dimension and Fact table in DWH**
  - As in OLAP/DWH we need to denormalized dimension tables and extract facts
  - we merged 'address' and 'city' table based on common column 'city_id' and create 'Dim_Address' table
  - delete 'add_id' column from 'customer' table and create 'Dim_Customer' table
  - leave 'merchant' table as it is and create 'Dim_Merchant' table
  - create new 'Dim_Date'
  - create new 'Fact_Transaction' table
    ```sql
    create table Dim_Address
    (
    	ADDRESS_ID		int		primary key,
    	STREET			varchar(40),
    	ZIP			int,
    	LATITUDE		float,
    	LONGITUDE		float,
    	CITY_NAME		varchar(30),
    	STATE			varchar(5),
    	CITY_POPULATION	        int,
            CENSUS_YEAR             char(4)
    )
    ```
    ```sql
    create table Dim_Customer
    (
    	CUSTOMER_ID	        int		primary key,
    	FIRST_NAME	        varchar(25),
    	LAST_NAME	        varchar(25),
    	CREDIT_CARD_NUMBER	varchar(25),
    	GENDER			char(1),
    	JOB			varchar(100),
    	DATE_OF_BIRTH		datetime
    )
    ```
    ```sql
    create table Dim_Merchant
    (
    	MERCHANT_ID	int,
    	MERCHANT_NAME	varchar(50),
    	CATEGORY	varchar(20),
    	MERCH_LATITUDE	float,
    	MERCH_LONGITUDE	float
    )
    ```
    ```sql
    create table Dim_Date
    (
            DATETIME_ID	int		primary key,
    	FULL_DATE	date,
    	DAY		int,
    	DAY_NAME	varchar(10),
    	WEEK		int,
    	MONTH		int,
    	MONTH_NAME	varchar(10),
    	QUARTER		int,
    	SEMESTER	int,
    	YEAR		int
    )
    ```
    ```sql
    create table Fact_Transaction
    (
    	CUSTOMER_ID		int,
    	ADDRESS_ID		int,
    	MERCHANT_ID		int,
    	DATETIME_ID		int,
    	AMOUNT			money,
    	IS_FRAUD		int
    )

- **Create Reference Tables/Views in DWH**
  - As there have so many changes in tables in DWH so, we need to have reference table/views, from where we will get the data.
  - We will use reference table because we will use more options in SSIS. By the way, below reference views also can be used for data fetching from CC_Transactions_Stage to CC_Transactions_DW.
  - reference view because 'address' table and 'city' table merged
    ```sql
    create view vw_Dim_Address
    as
    select add_id, street, zip, lat, long, city, state, city_pop
    from CC_Transactions_Stage.dbo.address_stage a full join CC_Transactions_Stage.dbo.city_stage c
    on a.city_id = c.city_id
    ```
  - reference view because one column deleted from 'customer' table
    ```sql
    create view vw_Dim_Customer
    as
    select cust_id, first, last, cc_num, gender, job, dob from CC_Transactions_Stage.dbo.customer_stage
    ```
  - 'merchant' table has no changes
  - reference view because need to add 'ADDRESS_ID' in 'Fact_Transaction' table of 'CC_Transactions_DW' server
    ```sql
    create view vw_TransactionID_to_AddID
    as
    select transaction_id, c.add_id
    from CC_Transactions_Stage.dbo.transactions_stage t
    join CC_Transactions_Stage.dbo.customer_stage c on t.cust_id = c.cust_id
    join CC_Transactions_Stage.dbo.address_stage a on c.add_id = a.add_id
    ```
  - reference view because need to add 'DATETIME_ID' in 'Fact_Transaction' table of 'CC_Transactions_DW' server
    ```sql
    create view vw_trans_date_trans_time_to_DateTimeID
    as
    select trans_date_trans_time, format(trans_date_trans_time, 'yyyyMMdd') datetimeid from CC_Transactions_Stage.dbo.transactions_stage
    ```
  - populate new date dimension table 'Dim_Date' for the first time and after that we will use different procedure
    ```sql
    declare @start datetime
    declare @end datetime = getdate()

    select @start = cast(min(trans_date_trans_time) as date) from CC_Transactions_Stage.dbo.transactions_stage

    while @start <= @end
    begin
    	insert into Dim_Date values
    	(
    		cast(format(@start, 'yyyyMMdd') as int),
    		cast(@start as date),
    		day(@start),
    		datename(dw, @start),
    		datepart(wk, @start),
    		month(@start),
    		datename(mm, @start),
    		case
    			when month(@start) in (1,2,3) then 1
    			when month(@start) in (4,5,6) then 2
    			when month(@start) in (7,8,9) then 3
    			when month(@start) in (10,11,12) then 4
    		end,
    		case
    			when month(@start) in (1,2,3,4,5,6) then 1
    			else 2
    		end,
    		year(@start)
    	)

    	set @start = dateadd(dd, 1, @start)
    end
    ```
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/d33fd2ca-1964-4500-a315-35a56fc6e4a6)
  - we have 4 dimension tables and 1 fact tables. So, we need to create 4 packages, named 'DWH_Load_Dim_Address', 'DWH_Load_Dim_Customer', 'DWH_Load_Dim_Merchant', 'DWH_Load_Dim_Date', 'DWH_Load_Fact_Transaction'.

- **Create SSIS Package for DWH Server**
  - open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/436e8a83-a63e-47f6-9b9d-dfc56051c244)

- **Data Loading to 'CC_Transactions_DW' Database from 'CC_Transactions_Stage' Database**
  - double click on 'DWH_Load_Dim_Address' SSIS Packages
  - drag 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'OLE DB Source'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'CC_Transactions_Stage' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose '[dbo].[address_stage]' from 'Name of the table or the view' -> Ok
  - drag 'Lookup'
    - to join tables
    - Look up on 'city_stage' table based on 'city_id' and get 'city', 'state', 'city_pop' columns. Reference 'ETL_Mapping.xlsx'
    - connect 'blue pipe' from 'OLE DB Source' to 'Lookup'
    - double click on it
    - choose 'Redirect rows to no match output' in 'Specify how to handle rows with no matching entries' in 'General'
    - in 'Connection' choose 'CC_Transactions_Stage' in 'OLE DB Connection Manager' and choose 'city_stage' in 'Use a table or a view'
    - in 'Columns' tab drag 'city_id' of 'Available Input Columns' on 'city_id' of 'Available Lookup Columns' and tick desired columns 'city', 'state' and 'city_pop'
    - change 'Output Alias' as 'city_lkp', 'state_lkp' and 'city_pop_lkp' -> Ok
  - drag 'Derived Column'
    - to add new column
    - connect 'blue pipe' from 'Lookup' to 'Derived Column'
    - choose 'Lookup Match Output' in 'Output' -> Ok
    - double click on it
    - enter 'Derived Column Name' as 'country_code_derived', choose 'Derived Column' as 'add as new column'
    - enter 'Expression' as '91' -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from 'Derived Column' to 'OLE DB Destination'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> again select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'CC_Transactions_DW' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view - fast load'
    - select '[dbo].[Dim_Address]' table in 'Name of the table or the view' which was created before 
    - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/3106e327-b802-4537-af72-d8eabc9cd4fe)
  - change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
    <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/d906c4c3-78f4-4dd2-b6f7-161b37b3aa72)
  - for 'DWH_Load_Dim_Date'
    - just drag a 'Execute SQL Task' and double click on it
    - in 'General' tab insert
      ```sql
      declare @start datetime = (select max(FULL_DATE) from Dim_Date)
      declare @end datetime = (select dateadd(dd, 1, max(FULL_DATE)) from Dim_Date)

      while @end <= getdate()
      begin
          insert into Dim_Date values
          (
          	cast(format(@end, 'yyyyMMdd') as int),
          	cast(@end as date),
          	day(@end),
          	datename(dw, @end),
          	datepart(wk, @end),
          	month(@end),
          	datename(mm, @end),
          	case
          		when month(@end) in (1,2,3) then 1
          		when month(@end) in (4,5,6) then 2
          		when month(@end) in (7,8,9) then 3
          		when month(@end) in (10,11,12) then 4
          	end,
          	case
          		when month(@end) in (1,2,3,4,5,6) then 1
          		else 2
          	end,
          	year(@end)
          )

          set @end = dateadd(dd, 1, @end)
      end
      ```
      in 'SQLStatement' for updating the 'Dim_Date' regularly
  - do the needful for 'DWH_Load_Dim_Customer', 'DWH_Load_Dim_Merchant', 'DWH_Load_Fact_Transaction' packages taking reference from 'ETL_Mapping.xlsx'
  - now, do the manual nad automatic Loggings for all the packages which was mentioned earlier.
  - now, if any update available in stage database we will load the same in DWH dimension tables only not in the fact tables, but one issue will occur i.e., again old data will load in dimension tables with new ones. So, we will use Slowly Changing Dimension (SCD) to negate the old data from copying with.
  - however, for incremental/delta loading we can use SCD, Lookup, Stored Procedure, Set Operators, Merge Command
  - remember, only Dimension tables need to be implemented with incrementa loading and for Fact tables need to be implemented with full loading.
    <br> &emsp;
  - Incremental loading using '**Lookup**'
    - we 'Lookup' on destination table and for unmatched data we will insert and for matched data we will update the same in destination table
    - double click on 'DWH_Load_Dim_Address'
    - drag 'Lookup' just before 'OLE DB Destination' to looking for new data or updated data
      - connect 'blue pipe' from 'Derived Column' to 'Lookup1'
      - double click on it
      - choose 'Redirect rows to no match output' in 'Specify how to handle rows with no matching entries' in 'General'
      - in 'Connection' choose 'CC_Transactions_DW' in 'OLE DB Connection Manager' and choose 'Dim_Address' in 'Use a table or a view'
      - in 'Columns' tab drag 'add_id' of 'Available Input Columns' on 'ADDRESS_ID' of 'Available Lookup Columns' -> Ok
    - now, for 'OLE DB Destination'
      - connect 'blue pipe' from Lookup1' to 'OLE DB Destination'
      - choose 'Lookup No Match Output' in 'Output' for inserting data -> Ok
    - now, drag 'OLE DB Command'
      - for updated data, to update data in destination table
      - connect 'blue pipe' from Lookup1' to 'OLE DB Command'
      - choose 'Lookup Match Output' in 'Output' for updating data -> Ok
      - double click on 'OLE DB Command'
      - in 'Connection Managers' tab, select 'destDB.CC_Transactions_DW' in 'Connection Managers'
      - in 'Component Properties' tab, in 'SqlCommand' enter
        ```sql
        update Dim_Address
        set STREET= ?, ZIP = ?, LATITUDE = ?, LONGITUDE = ?, CITY_NAME = ?, STATE = ?, CITY_POPULATION = ?, CENSUS_YEAR = ?
        where ADDRESS_ID = ?
        ```
    - in 'Column Mappings' tab, map 'Input Column' and 'Destination Column' -> Ok
  - But the problem with 'OLE DB Command' is that it does not verify whether there have any changes in the matched data or not, it just takes all the data and updating or over writing the same. So, to identify any change in source data we need to use another 'Lookup' before 'OLE DB Command' and identify the actual data where updation is required.
    - in 'Dim_Address' table only the 'CITY_POPULATION' and 'CENSUS_YEAR' columns could be updated over time but the other columns like 'STREET', 'ZIP', 'LATITUDE', 'LONGITUDE', 'CITY_NAME', 'STATE' cannot updatd
    - drag 'Lookup' just before 'OLE DB Command'
    - connect previous 'Lookup' to 'Lookup 2'
    - double click on 'Lookup 2'
    - choose 'Redirect rows to no match output' in 'Specify how to handle rows with no matching entries' in 'General'
    - in 'Connection' choose 'CC_Transactions_DW' in 'OLE DB Connection Manager' and choose 'Dim_Address' in 'Use a table or a view'
    - in 'Columns' connect ADDRESS_ID, CITY_POPULATION, CENSUS_YEAR only -> Ok
    - connect 'blue pipe' from 'Lookup 2' to 'OLE DB Command'
    - choose 'Lookup No Match Output' in 'Output' for inserting updated data -> Ok
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/d501f57b-9ce7-403f-addb-0b141c34d2ad)
  - Now, we cannot load all the tables at once, we need to load tables where PKs are present then we can load table where FKs are present.
    - now, create one more package named 'DWH_load_tables.dtsx'
    - double click on it
    - drag 'Execute Package Task' and double click on it
    - in 'Package' tab choose the first package in 'PackageNameFromProjectReference' -> Ok
    - again drag another 'Execute Package Task 1'
    - connect 'green pipe' from ' Execute Package Task ' to ' Execute Package Task 1'
    - double click on it and in 'Package' tab choose the second package in 'PackageNameFromProjectReference' -> Ok
    - do the same things till last package
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/6678dd1a-5589-4e19-ba0f-7c32ce9ef7cb)
  <br> &emsp;
  - SSIS Performance
    - Now, due to 'OLE DB Command' in 'Data Flow' the task becomes very slow because it performs row by row operation, to overcome this we will update the data in 'Control Flow' using 'Execute SQL Task' because it performs batch operation.
    - open 'DWH_Load_Dim_Address.dtsx'
    - delete the 'OLE DB Command' and drag 'OLE DB Destination 1'
      - connect the 'blue pipe' from 'Lookup 2' to 'OLE DB Destination 1'
      - choose 'Lookup No Match Output' as 'Output' -> Ok
      - double click on ' OLE DB Destination 1'
      - in 'connection Manager' select 'CC_Transactions_DW' in 'OLE DB Connection manager'
      - select 'Data access mode' as 'Table or view - fast load'
      - select 'New' in 'Name of the table or the view'
      - change table name to 'Temp_Address' and change data type if needed -> Ok
      - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
        <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/a2c5d667-e500-4dcb-a1bf-55f91ff80b1e)
    - now, in 'Control Flow' drag 'Execute SQL Task'
      - double click on it
      - in 'General' select 'OLE DB' in 'ConnectionType' and 'CC_Transactions_DW' in 'Connection'
      - add
        ```sql
        update Dim_Address
        set CITY_POPULATION = t.city_pop_lkp, CENSUS_YEAR = t.CENSUS_YEAR_der
        from Dim_Address d join Temp_Address t on d.ADDRESS_ID = t.add_id
        go

        truncate table Temp_Address
        go
        ```
        in 'SQLStatement' -> Ok -> Ok
        <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/412d0dfd-d27a-46ba-81f2-8631518c4621)
  <br> &emsp;
  - Incremental loading using '**Slowly Changing Dimension**'
    - double click on 'DWH_Load_Dim_Address'
    - drag 'Slowly Changing Dimension' after 'Derived Column'
      - connect 'blue pipe' from 'Derived Column' to 'Slowly Changing Dimension'
      - double click on it
      - Next -> select '[dbo].[Dim_Address]' in 'Table or view'
      - select proper 'Input Columns' and 'Dimension Columns' and then choose the 'Business key' column -> Next
        <br> **Remember**
      	<br> Fixed Attribute (Type 0) -> data that will not update or change anymore like, Name, Gender, DOB etc.
      	<br> Changing Attribute (Type 1) -> data that will update or change and we can overwrite the same in database like, the City you staying etc.
      	<br> Historical Attribute (Type 2) -> data that will update or change and we need to maintain the historic data in database like, Salary increasing every year etc. 
      - now select dimension column in 'Dimension Columns' and select 'Changing Attribute' in 'Change Type' -> Next -> Next -> Next -> Finish
      - and it will create everything for you
        <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/8c83032a-9a56-4a53-9599-73ba63b757d9)
        <br> **Remember**
        <br> if Historical Attribute (Type 2) is there then we cannot have Primary Key in database because Type 2 save duplicate data in Primary Key column
      - just remember performance wise 'Slowly Changing Dimension' is not so good because it uses 'OLE DB Command' for data loading and as we know 'OLE DB Command' perform row by row wise.
