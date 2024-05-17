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
    <br> &emsp;
  - Incremental loading using '**Stored Procedure**'
    - load data from 'Credit_Card_Transactions' server to 'CC_Transactions_Stage' server
      - for example we are going to load 'address_stage' table in 'CC_Transactions_Stage' server from 'address' table in 'Credit_Card_Transactions' server
      - now open SQL Server Management Studio
      - creating a stored procedure in 'CC_Transactions_Stage' server that will load data from 'address' table to 'address_stage' table
        ```sql
        create proc usp_load_address_stage
        as
        begin
        	-- delete all data from stage table
        	delete from CC_Transactions_Stage.dbo.address_stage

        	-- insert data to stage table from oltp server
        	insert into CC_Transactions_Stage.dbo.address_stage
        	select * from Credit_Card_Transactions.dbo.address
        end
        ```
      - calling the stored procedure `exec usp_load_address_stage`
    - now load 'Dim_Address' in 'CC_Transactions_DW' from 'address_stage' in 'CC_Transactions_Stage' and few columns will add and delete in final dimension table
    - *also note that in data warehouse we will use incremental loading not full loading*
      - creating a stored procedure in 'CC_Transactions_DW' server that will load data incrementally from 'address_stage' to 'Dim_Address'
        ```sql
        create proc usp_incr_load_dim_address
        as
        begin
        	--Incremental Insert
        		-- select columns that we need for dimension table
        		insert into Dim_Address
        		select a.add_id, a.street, a.zip, a.lat, a.long, c.city, c.state, c.city_pop, 2020 as census_year
        		-- joining two stage table to get 'city_name', 'state', 'city_pop' column
        		from CC_Transactions_Stage.dbo.address_stage a
        		join CC_Transactions_Stage.dbo.city_stage c on a.city_id = c.city_id
        		-- left joining with 'Dim_Address' column to know about new data
        		left join CC_Transactions_DW.dbo.Dim_Address da on a.add_id = da.ADDRESS_ID
        		where da.ADDRESS_ID is null

        	-- Incremental Update
        		update da
        		set da.STREET = a.street, da.ZIP = a.zip, da.LATITUDE = a.lat, da.LONGITUDE = a.long,
        		da.CITY_NAME = c.city, da.STATE = c.state, da.CITY_POPULATION = c.city_pop
        		from CC_Transactions_Stage.dbo.address_stage a
        		-- joining two stage table to get 'city_name', 'state', 'city_pop' column
        		join CC_Transactions_Stage.dbo.city_stage c on a.city_id = c.city_id
        		-- joining with 'Dim_Address' column to know about the updated data
        		join CC_Transactions_DW.dbo.Dim_Address da on a.add_id = da.ADDRESS_ID
        		where a.street != da.STREET or a.zip != da.ZIP or a.lat != da.LATITUDE or a.long != da.LONGITUDE
        		or c.city != da.CITY_NAME or c.state != da.STATE or c.city_pop != da.CITY_POPULATION
        end
        ```
      - calling the stored procedure `exec usp_incr_load_dim_address`
    - major drawback in 'Stored Procedure' is to maintain the code and writing the complex code for more columns.
  <br> &emsp;
  - Incremental loading using '**Merge Statement**'
    - A 'Merge Statement' is a SQL statement that performs INSERT, UPDATE, and DELETE operations based on the existence of rows matching the selection criteria in the target table.
      - now load 'Dim_Address' table in 'CC_Transactions_DW' from 'address_stage' table in 'CC_Transactions_Stage' and few columns will add and delete in final dimension table
      - creating a 'merge statement' 'CC_Transactions_DW' server in that will load data incrementally from 'address_stage' to 'Dim_Address'
        ```sql
        create proc usp_merge_Dim_Address
        as
        begin
        	-- inserting source table into temp table
        	select a.add_id, a.street, a.zip, a.lat, a.long, c.city, c.state, c.city_pop, 2020 as census_year
        	into #temp
        	from CC_Transactions_Stage.dbo.address_stage a join CC_Transactions_Stage.dbo.city_stage c
        	on a.city_id = c.city_id

        	merge Dim_Address as d -- destination table
        	using #temp as s -- source table
        	on d.ADDRESS_ID = s.add_id

        	when not matched by target -- Insert
        	then
        	insert (ADDRESS_ID, STREET, ZIP, LATITUDE, LONGITUDE, CITY_NAME, STATE, CITY_POPULATION, CENSUS_YEAR)
        	values(s.add_id, s.street, s.zip, s.lat, s.long, s.city, s.state, s.city_pop, s.census_year)

        	when matched and s.street <> d.STREET or s.zip <> d.ZIP or s.lat <> d.LATITUDE or s.long <> d.LONGITUDE
        	or s.city <> d.CITY_NAME or s.state <> d.STATE or s.city_pop <> d.CITY_POPULATION or s.census_year <> d.CENSUS_YEAR  -- Update
        	then
        	update
        	set d.street = s.street, d.ZIP = s.zip, d.LATITUDE = s.lat, d.LONGITUDE = s.long,
        	d.CITY_NAME = s.city, d.STATE = s.state , d.CITY_POPULATION = s.city_pop, d.CENSUS_YEAR = s.census_year;
        end
        ```
      - remember we will not delete anything here because we will use incremental loading.
  <br> &emsp;
  - Why '**Full Loading/Partial Loading**' to the Fact Table
    - OLTP will grow in size over time. Generally OLTP is smaller in size. So, we need to load complete fact data from OLTP to OLAP and then delete all data in OLTP fact table.
    - Generally we need to historical fact data to analyze. So, we will perform only Insert operation in fact table of OLAP database.
    - In fact tables there have no primary key. So, we cannot perform incremental loading.
    - In partial loadind for example, we need to maintain 3 month sales data in OLTP data base so, we will transfer 1 month data to OLAP after every montn end.
    - So, we can use below mention SSIS package for full loading.
      <br> ![image](https://github.com/sumanndass/Credit_Card_Transactions_MSBI/assets/156992689/d75743ed-54e1-4026-abfe-9123fbe7a076)


### SSIS Deployment
- DEV tasks
  - Create a SSIS Package / Perform Unit Testing
  - Implement Logging in SSIS
    - Extensions -> SSIS -> Logging -> tick the 'Containers' from left box -> click add 'SSIS log provider for SQL Server' and 'SSIS log provider for Windows Event Log' in 'Providers and Logs' tab -> tick previous two log below -> select a server for 'SSIS log provider for SQL Server' in 'Configuration' -> in 'Details' tab selects 'OnError' & 'OnPostExecute' & 'OnTaskFailed' -> OK
    - Event Handler (Customized Logging)
  - Build and CheckIn the Code to VSTF / TFS
    - right click on project name and click on 'Build' to find any issue with the project -> now go to project path 'D:\...\...\...\SSIS\CC_Transaction_DW_SSIS\CC_Transaction_DW_SSIS\bin\Development' there must be an .ispac file, this file having all the packages -> and upload this .ispac file to VSTF/TFS server -> now you have to give deployment guide as .txt or .docx file like
      <br> Project Name:
      <br> Author:
      <br> Purpose:
      <br> Steps:
      1. go to VSTF server and download <name> folder and its content
      2. go to DWH and open SQL Server Management Studio
      3. connect to server
      4. right click on integration service catalogs and choose create a catalog
      5. right click on SSISDB and choose folder
      6. name = 'Credit_Card_Transactions' and ok
      7. double click on .ispac file
      8. Next
      9. Next
      10. choose 'SSIS in SQL Server' and Next
      11. select Server
      12. click on 'Connect'
      13. Browse for the folder
      14. Next
      15. Deploy
      16. Close
- DBA tasks
  - Download all files from VSTF/TFS server to DWH server
  - Create Integration Services Catalogs in SQL Server of DWH
    - open SSMS -> right click on 'Integration Service Catalogs' -> click on 'Create Catalog' -> click on 'Enable CLR Integration' -> enter password '12345' or anything you want -> OK -> now click on SSISDB and click on 'Create Folder' -> put name -> OK
  - Deploy the package using ispac file
    - double click on .ispac file -> next -> next -> choose 'SSIS in SQL Server' and 'Next' -> put 'Server Name' as '.' -> click on 'Connect' -> now 'Browse' for the folder which was created in SSMS Integration Service Catalogs -> select the folder and change the name as you wish in 'Path' -> Next -> click on 'Deploy' -> Close
    - now go to SSMS and refresh 'Integration Service Catalogs' -> and all the packages will be there in the folder which was created earlier -> now create a document what the packages are doing
    - in real life all the servers will be different so in that case we need to configure the connections -> go to SSMS and open the folder in 'Integration Service Catalogs' -> right click on packages and click on 'Configure' -> go to 'Connection Manager' tab -> change server name -> click on 3 dots -> click on 'Edit values' and enter server address -> ok -> ok -> right click on packages and click on 'Execute' -> ok, this will execute the task -> click 'Yes' to report the task -> report will generate and also you can find the SQL Server logging in System Table in Server (logged database -> Tables -> System Tables)
  - Schedule a Package Using SQL Server Jobs
    - to schedule task -> go to 'SQL Server Agent' and start it by right click on it -> expand it and right click on 'Jobs' -> click on 'New Job' -> put job name like 'Dim_Address_Load_Job' in 'General' tab -> click on 'Steps' tab -> click on 'New' -> put 'Step Name' like 'attach_package' -> select type 'SQL Server Integration Services Packages' -> now in below 'Package' tab choose 'SSIS Catalog' in 'Package source' -> put 'Server' -> now click on 3 dots in 'Package' and choose one package -> ok -> now go to 'Schedule' tab -> click on 'New' -> put 'Name' -> choose 'Occurs' as daily, weekly, monthly -> click on 'Occurs once at' and put time -> ok -> ok -> now task will automatically run at 01:00AM daily or you can run the same at any time -> right click on the task and click on 'Start job at step' -> and you can watch the logging, by double clicking on 'Job Activity Monitor'


### SSAS Create Cube
- **Create SSAS Project**
  - Create SSAS project using 'Analysis Services Multidimensional Project' in DevEnv
    **Remember**
    <br> Multidimensional - for more data, save in disk, slow performace, use MDX
    <br> Tabular - for few GB of data, save in RAM, fast performace, use DAX

- **Create Data Source**
  - right click on 'Data Sources'
  - select 'New Data Source'
  - Next
  - choose 'Create a data source based on an existing or new connection'
  - click on 'new'
  - change provider from 'Native OLE DB' to 'Microsoft OLE DB Driver for SQL Server'
  - enter 'Server or file name' or '.'
  - select or enter a database name in 'Initial catalog' like here 'CC_Transactions_DW' server
  - check 'Test Connection' - ok - ok
  - next
  - select the 'Use the service account'
  - next
  - enter data source name 'ds_CC_Transactions_DW'
  - finish
  
- **Create Data Source View**
  - right click on 'Data Source Views'
  - select ‘New Data Source View’
  - next
  - select data source from 'Relational Data Sources'
  - next
  - select fact tables first
  - click on ‘Add Related Tables’ to select all other tables related to fact tables or manually choose required tables
  - next
  - enter name 'dsv_CC_Transactions_DW'
  - finish
  - double click on 'dsv_CC_Transactions_DW'
  - match the relationships if needed
  - basically, you need to create multiple 'data source view' as per your requirements and on the basis of 'data source view' you will create cubes

- **Create Cube**
  - right click on 'Cubes'
  - select 'New Cube'
  - next
  - choose 'Use existing tables’
  - next
  - now select only fact/measure tables
  - next
  - now select only fact/measures as per your requirement or select all measures
  - next
  - now select dimensions as per your requirement or select all dimensions
  - next
  - now enter 'Cube name:' as 'cube_CC_Transactions_DW'
  - finish

- **Build Dimension**
  - Dimensions will automatically create when cube is formed
    - but to view new items -> double click on dimensions in solution explorer -> drag items from 'Data Source View' to 'Attributes'
    - if hierarchy is needed then drag items from 'Attributes' to 'Hierarchies' -> enter name of the hierarchy -> in 'Attribute Relationship' tab arrange attributes from smallest to highest -> save -> ok -> if we need, we can create more hierarchies
      - if any issue creates then double click on the selected dimension -> right click on the selected 'Attributes' -> go to 'Properties', in 'KeyColumns' select its prior attributes -> arrange the same from smallest to highest -> Ok -> in 'NameColumn' choose the selected attributes -> ok
    - now again rebuild and process the cube if any changes are done
    - now go to browser and refresh or reconnect the cube
  
- **Create Calc**

- **Deploy the Cube**
  - right click on project in solution explorer
  - go to 'Properties'
  - select 'Deployment'
  - enter 'Server' name or '.'
  - enter 'Database' name or 'cube_bank_dw'
  - Apply - Ok
  - now again right click on project in solution explorer
  - select 'Build'
  - now again right click on project in solution explorer
  - select 'Process' - yes
  - process options ‘process full’
  - run - close - close
  - double click on the cube 
  - browser
  - reconnect
- click on 'Browser' (to see all the aggregations)

- **Testing the Cube - Excel**
  - now you can use this cube. Like: below example
    - open excel -> data -> from database -> from analysis services -> enter server name or . -> log on credential 'use windows authentication' -> next -> select cube -> next -> finish -> ok -> now do anything as your wish

