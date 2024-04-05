# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)
- [SSIS for Stage Database (SQL Server Integration Service)](#ssis-for-stage-database-sql-server-integration-service)
- [SSIS for Data Warehouse Database (SQL Server Integration Service)](#ssis-for-data-warehouse-database-sql-server-integration-service)
- [SSIS Deployment](#ssis-deployment)
- [SSAS Create Cube](#ssas-create-cube)

### Overview
- This is a daily bank operation project.
- Assume daily few data are storing in 'bank' database and the tables are 'account_table' and 'transaction_table'.
- Also, few data are storing in 'product_doc' excel file.
- Also, few data are storing in 'branch_staff_doc' excel file and sheets are 'branch' and 'staff'.
- We need to load the data from 'bank' and 'bank_doc' to 'bank_stage' database and perform ETL operation on the same.

### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - Cube, Pre aggregating values
- SSRS -

### SSIS for Stage Database (SQL Server Integration Service)
- **create a stage database named 'bank_stage' (where ETL will happen)**
  ```sql
  create database bank_stage
  ```

- **use the 'bank_stage' database**
  ```sql
  use bank_stage
  ```

- **Create SSIS Project**
  - Create SSIS project using 'Integration Service Project' in DevEnv

- **Create SSIS Package for Stage Server**
  - open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
    - we have 2 database table and 2 excel file, one excel workbook has two sheets. So, we need to create 5 packages, named 'account_db', 'transaction_db', 'region_db', 'branch_doc', 'staff_doc', 'product_doc'.

	**Remember**
  <br> Data Flow - ETL Activities
	<br> Control Flow - Non-ETL Activities

- **Data Loading to 'bank_stage' Database from 'bank' Database**
  - double click on 'account_db' SSIS Packages
  - drag 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'OLE DB Source'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'bank' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose '[dbo].[account_table]' from 'Name of the table or the view' -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from source to destination
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'bank_stage' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'New' in 'Name of the table or the view'
    - change table name to 'account_stage' and change data type if needed - Ok
    - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
  - change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  - stage table always needs fresh data
  - so, drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
    - add SQL truncate command (`truncate table account_stage`) to delete all old data from stage whenever new data comes -> Ok
    - connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
    - 'Start' the project
    - do the same for 'transaction_db' and 'region_db' package

- **Data Loading to 'bank_stage' Database from 'Excel' Document**
  - double click on 'branch_doc' SSIS Packages
  - 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'Excel Source'
    - double click on it
    - in 'connection Manager' select 'New'
    - find the 'excel' document path and choose the same and select proper 'Excel version' -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose 'branch$' from 'Name of the Excel sheet' -> Ok
  - drag 'Data Conversion'
    - connect 'blue pipe' from 'Excel Source' to 'Data Conversion'
    - double click on it
    - select the 'Available Input Columns'
    - change the 'Data Type' and 'Length' -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from 'Data Conversion' to 'OLE DB Destination'
    - double click on it
    - in 'connection Manager' select 'dest.bank_stage' (which was saved earlier) in 'OLE DB Connection manager'
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'New' in 'Name of the table or the view'
    - change table name to 'branch_stage' and change data type if needed -> Ok
    - now click on 'Mappings' choose proper 'Input Column' and 'Destination Column' -> Ok
  - change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  - stage table always needs fresh data
  - so, drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
    - add SQL truncate command (`truncate table account_stage`) to delete all old data from stage whenever new data comes -> Ok
    - connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
    - 'Start' the project
    - do the same for 'staff_doc' and 'product_doc' packages

- **SSIS Logging (To know about the status of the successful loadings)**
  - create a SSIS logging table named 'ssis_log' (where SSIS status will store)
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
    - open 'account_db' package and drag 'Execute SQL Task' in 'Control Flow'
    - double click on it
    - in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
    - add 'SQLStatement' command (`insert into ssis_log values(?, getdate(), ?, 'Success...')`)
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
    - connect 'green pipe' from 'Data Flow Task' to new 'Execute SQL Task 1'
    - 'Start' the project
    - do the same for 'transaction_db', 'branch_doc', 'staff_doc', 'product_doc' packages

- **SSIS Failure Logging (To know about the failure happened in loading)**
  - double click on desired package -> go to 'Extension' -> 'SSIS' -> 'Logging'
  - choose all 'Containers:' -> 'Add' 'SSIS log provider for SQL Server', tick the same and choose the destination server 'bank_stage' in 'Configuration' -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in system tables in the selected database by using command `sql select * from sysssislog`
  - again, choose all 'Containers:' -> 'Add' 'SSIS log provider for Windows Event Log', tick the same -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  - Find the error logs in 'Windows Event Viewer' -> 'Windows Logs' -> 'Application'
  - Set this task for other packages also

### SSIS for Data Warehouse Database (SQL Server Integration Service)
- **Create Data Warehouse Database**
  ```sql
  create database bank_dw
  go

  use bank_dw
  go
  ```

- **Create Dimension and Fact table in DWH**
  - As in OLAP/DWH we need to denormalized dimension tables and extract facts, so
  - we merged account_table and product_table
  - we merged branch_table and region_table
  - we merged transaction_table and staff_table
  - we created a new dimension date table
  - we created a new dimension location table
  - we extracted facts from account table and created a new fact table
  - we extracted facts from transaction table and created a new fact table

  ```sql
  create table dim_date
  (
  	id		int		primary key	identity(1,1),
  	date_id		int		unique,
  	full_date	datetime,
  	year_no		int,
  	sem_no		int,
  	qtr_no		int,
  	month_no	int,
  	month_name	varchar(10),
  	week_no		int,
  	day_no		int,
  	day_name	varchar(10)
  )
  ```
  ```sql
  create table dim_location
  (
  	loc_id	int			primary key,
  	country	varchar(50),
  	state	varchar(30) unique,
  	city	varchar(30)
  )
  ```
  ```sql
  create table dim_branch
  (
  	br_id		varchar(4)	primary key,
  	br_name		varchar(20),
  	br_add		varchar(70),
  	br_state	varchar(30),
  	br_zipcode	char(6),
  	reg_id		int,
  	reg_name	char(6)
  )
  ```
  ```sql
  create table dim_account
  (
  	acc_id			int		primary key,
  	cust_name		varchar(30),
  	cust_add		varchar(70),
  	cust_state		varchar(30),
  	cust_zipcode		varchar(6),
  	br_id			varchar(4),
  	prod_id			varchar(2),
  	prod_name		varchar(20),
  	status			varchar(1),
  	country_code		int
  )
  ```
  ```sql
	  create table dim_transaction
  (
  	tran_id		int		primary key,
  	acc_id		int,
  	br_id		varchar(4),
  	txn_type	varchar(3),
  	chq_no		varchar(6),
  	chq_date	datetime,
  	staff_id	int,
  	staff_name	varchar(40),
  	designation	varchar(2)
  )
  ```
  ```sql
  create table fact_account
  (
  	acc_id			int,		
  	doo_date_id		int		foreign key references dim_date(date_id),
  	clr_bal			money,
  	unclr_bal		money
  )
  ```
  ```sql
  create table fact_transaction
  (
  	tran_id			int,		
  	dot_date_id		int		foreign key references dim_date(date_id),
  	txn_amt			money
  )
  ```

- **Create Reference Tables/Views in DWH**
  - As there have so many changes in tables in DWH so, we need to have reference table/views, from where we will get the data.
  - We will use referencee table becuase we will use more options in SSIS. By the way, below reference views also can be used for data fetching from bank_stage to bank_dw.
  
  - reference view because account_table and product_table merged
    ```sql
    create view vw_ref_dim_account
    as
    select a.acc_id, a.cust_name, a.cust_add, a.cust_state,
    a.cust_zipcode, a.br_id, a.prod_id, p.prod_name, a.status
    from bank.dbo.account_table a join bank.dbo.product_table p on a.prod_id = p.prod_id
    ```
  - reference view because branch_table and region_table merged
    ```sql
    create view vw_ref_dim_branch
    as
    select b.br_id, b.br_name, b.br_add, b.br_state, b.br_zipcode, r.reg_id, r.reg_name
    from bank.dbo.branch_table b join bank.dbo.region_table r on b.reg_id = r.reg_id
    ```
  - reference view because transaction_table and staff_table merged
    ```sql
    create view vw_ref_dim_transaction
    as
    select t.tran_id, t.acc_id, t.br_id, t.txn_type, t.chq_no, t.chq_date, t.staff_id, s.staff_name, s.designation
    from bank.dbo.transaction_table t join bank.dbo.staff_table s on t.staff_id = s.staff_id
    ```
  - populate new date dimension table dim_date
    ```sql
    declare @startdate datetime
    declare @enddate datetime = getdate()
    select @startdate = cast(min(doo) as date) from bank.dbo.account_table

    while @startdate <= @enddate
    begin
    	insert into dim_date values
    	(
  		cast(format(@startdate, 'ddMMyyyy') as int),
  		@startdate,
  		year(@startdate),
  		case
  			when month(@startdate) in (1, 2, 3, 4, 5, 6) then 1
  			else 2
  		end,
  		case
  			when month(@startdate) in (1, 2, 3) then 1
  			when month(@startdate) in (4, 5, 6) then 2
  			when month(@startdate) in (7, 8, 9) then 3
  			when month(@startdate) in (10, 11, 12) then 4
  	   	end,
    		month(@startdate),
  	   	datename(mm, @startdate),
  	   	datepart(ww, @startdate),
  	   	day(@startdate),
  	   	datename(dw, @startdate)
  	)

  	set @startdate = dateadd(dd, 1, @startdate)
    end
    ``` 
  - populate new dimension location table dim_location
    ```sql
    insert into dim_location values
    (1, 'India', 'Andhra Pradesh', 'Visakhapatnam'),
    (2, 'India', 'Arunachal Pradesh', 'Itanagar'),
    (3, 'India', 'Assam', 'Guwahati'),
    (4, 'India', 'Bihar', 'Patna'),
    (5, 'India', 'Chhattisgarh', 'Raipur'),
    (6, 'India', 'Delhi', 'New Delhi'),
    (7, 'India', 'Goa', 'Vasco da Gama'),
    (8, 'India', 'Gujarat', 'Ahmedabad'),
    (9, 'India', 'Haryana', 'Faridabad'),
    (10, 'India', 'Himachal Pradesh', 'Shimla'),
    (11, 'India', 'Jharkhand', 'Jamshedpur'),
    (12, 'India', 'Karnataka', 'Bengaluru'),
    (13, 'India', 'Kerala', 'Kochi'),
    (14, 'India', 'Madhya Pradesh', 'Indore'),
    (15, 'India', 'Maharashtra', 'Mumbai'),
    (16, 'India', 'Manipur', 'Imphal'),
    (17, 'India', 'Meghalaya', 'Shillong'),
    (18, 'India', 'Mizoram', 'Aizawl'),
    (19, 'India', 'Nagaland', 'Dimapur'),
    (20, 'India', 'Odisha', 'Bhubaneswar'),
    (21, 'India', 'Punjab', 'Ludhiana'),
    (22, 'India', 'Rajasthan', 'Jaipur'),
    (23, 'India', 'Sikkim', 'Gangtok'),
    (24, 'India', 'Tamil Nadu', 'Chennai'),
    (25, 'India', 'Telangana', 'Hyderabad'),
    (26, 'India', 'Tripura', 'Agartala'),
    (27, 'India', 'Uttar Pradesh', 'Lucknow'),
    (28, 'India', 'Uttarakhand', 'Dehradun'),
    (29, 'India', 'West Bengal', 'Kolkata')
    ```
  - we have 7 dimension tables and 2 fact tables. So, we need to create 7 packages, named 'DWH_Load_dim_branch', 'DWH_Load_dim_account', 'DWH_Load_dim_transsaction', 'DWH_Load_fact_account', 'DWH_Load_fact_transaction' and 2 derived dimension tables ('dim_date' and 'dim_location') are already loaded.

- **Create SSIS Package for DWH Server**
  - open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages

- **Data Loading to 'bank_dw' Database from 'bank_stage' Database**
  - double click on 'DWH_Load_dim_account' SSIS Packages
  - drag 'Data Flow Task' in 'Control Flow' section
  - double click on 'Data Flow Task'
  - drag 'OLE DB Source'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'bank_stage' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view'
    - choose '[dbo].[account_stage]' from 'Name of the table or the view' -> Ok
  - drag 'Lookup'
    - Look up on 'product_stage' table based on 'prod_id' and get 'prod_name', reference 'ETL_Mapping _Doc.xlsx'
    - connect 'blue pipe' from 'OLE DB Source' to 'Lookup'
    - double click on it
    - choose 'Redirect rows to no match output' in '﻿Specify how to handle rows with no matching entries' in 'General'
    - in 'Connection' choose 'bank_stage' in 'OLE DB Connection Manager' and choose 'product_stage' in 'Use a table or a view'
    - in 'Columns' drag 'prod_id' of 'Available Input Columns' on 'prod_id' of 'Available Lookup Columns' and tick desired column 'prod_name'
    - change 'Output Alias' as 'prod_name_lkp' -> Ok
  - drag 'Derived Column'
    - connect 'blue pipe' from 'Lookup' to 'Derived Column'
    - choose 'Lookup Match Output' in 'Output' -> Ok
    - double click on it
    - enter 'Derived Column Name' as 'country_code_derived', choose 'Derived Column' as 'add as new column'
    - enter 'Expression' as '91', choose 'Data Type' as required -> Ok
  - drag 'OLE DB Destination'
    - connect 'blue pipe' from 'Derived Column' to 'OLE DB Destination'
    - double click on it
    - in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> again select 'New' -> Ok
    - select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
    - put 'Server or file name' as '.' -> select database name 'bank_dw' in 'Initial catalog' -> Ok -> Ok
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'New' in 'Name of the table or the view'
    - change table name to 'dim_account' and change data type if needed -> Ok
    - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
  - change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  - do the same for 'DWH_Load_dim_branch', 'DWH_Load_dim_transsaction', 'DWH_Load_fact_account', 'DWH_Load_fact_transaction' packages taking reference from 'ETL_Mapping_Doc.xlsx'
  - now, if any update available in stage database we will load the same in DWH dimension tables only not in the fact tables, but one issue will occur i.e., again old data will load in dimension tables with new ones. So, we will use Slowly Changing Dimension (SCD) to negate the old data from copying with.
  - however, for incremental/delta loading we can use SCD, Lookup, Stored Procedure, Set Operator, Merge Command
  
  - Incremental loading using 'Lookup'
  - we 'Lookup' on destination table and for unmatched data we will insert and for matched data we will update the same in destination table
  - double click on 'DWH_Load_dim_account'
  - drag 'Lookup' just before 'OLE DB Destination' i.e., data loading
    - connect 'blue pipe' from 'Derived Column' to 'Lookup1'
    - double click on it
    - choose 'Redirect rows to no match output' in 'Specify how to handle rows with no matching entries' in 'General'
    - in 'Connection' choose 'bank_dw' in 'OLE DB Connection Manager' and choose 'dim_account' in 'Use a table or a view'
    - in 'Columns' drag 'acc_id' of 'Available Input Columns' on 'acc_id' of 'Available Lookup Columns' and tick desired column 'prod_name' -> Ok
    - now, drag 'OLE DB Destination'
    - connect 'blue pipe' from Lookup1' to 'OLE DB Destination'
    - choose 'Lookup No Match Output' in 'Output' for inserting data -> Ok
    - double click on 'OLE DB Destination'
    - in 'connection Manager' select 'bank_dw' in 'OLE DB Connection manager'
    - select 'Data access mode' as 'Table or view - fast load'
    - select 'dim_account' in 'Name of the table or the view'
    - now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
    - now, drag 'OLE DB Command'
    - connect 'blue pipe' from Lookup1' to 'OLE DB Command'
    - choose 'Lookup Match Output' in 'Output' for updating data -> Ok
    - double click on 'OLE DB Command'
    - in 'Connection Manager' select 'bank_dw' in 'Connection Managers'
    - in 'SqlCommand' enter
      ```sql
      update dim_account
      set cust_name = ?, cust_add = ?, cust_state = ?, cust_zipcode = ?, prod_name = ?,	status = ?
      where acc_id = ?
      ``` in 'Component Properties'
    - in 'Column Mappings' map 'Input Column' and 'Destination Column' -> Ok
  - But the problem with 'OLE DB Command' is that it does not verify whether there have any changes in the matched data or not, it just takes all the data and updating or over writing the same. So, to identify any change in source data we need to use another 'Lookup' before 'OLE DB Command' and identify the actual data where updation is required.
    - drag 'Lookup' just before 'OLE DB Command'
    - connect 'blue pipe' from 'Lookup 2' to 'OLE DB Command'
    - choose 'Lookup No Match Output' in 'Output' for inserting data -> Ok
    - double click on 'Lookup 2'
    - choose 'Redirect rows to no match output' in 'Specify how to handle rows with no matching entries' in 'General'
    - in 'Connection' choose 'bank_dw' in 'OLE DB Connection Manager' and choose 'dim_account' in 'Use a table or a view'
    - in 'Columns' connect 'acc_id' to 'acc_id', 'cust_name' to 'cust_name', 'cust_add' to 'cust_add', 'cust_state' to 'cust_state', 'cust_zipcode' to 'cust_zipcode', 'prod_name_lkp' to 'prod_name', 'status' to 'status' -> Ok
  - Now, we cannot load all the tables at once, we need to load tables where PKs are present then we can load table where FKs are present.
    - now, create one more package named 'DWL_load_tables.dtsx'
    - double click on it
    - drag 'Execute Package Task' and double click on it
    - in 'Package' choose the first package in 'PackageNameFromProjectReference' -> Ok
    - again drag another 'Execute Package Task 1'
    - connect 'green pipe' from ' Execute Package Task ' to ' Execute Package Task 1'
    - double click on it and in 'Package' choose the second package in 'PackageNameFromProjectReference' -> Ok
    - do the same thing till last package
  
  <br> -> SSIS Performance
  <br> -> Now, due to 'OLE DB Command' in 'Data Flow' the task becomes very slow because it performs row by row operation, to overcome this we will update the data in 'Control Flow' using 'Execute SQL Task' because it performs batch operation.
      	<br> &emsp; -> open 'DWH_Load_dim_account.dtsx'
      	<br> &emsp; -> delete the 'OLE DB Command' and drag 'OLE DB Destination 1'
      	<br> &emsp; -> connect the 'blue pipe' from 'Lookup 2' to 'OLE DB Destination 1'
      	<br> &emsp; -> choose 'Lookup No Match Output' as 'Output' -> Ok
      	<br> &emsp; -> double click on ' OLE DB Destination 1'
   	<br> &emsp; -> in 'connection Manager' select 'banl_dw' in 'OLE DB Connection manager'
  	<br> &emsp; -> select 'Data access mode' as 'Table or view - fast load'
  	<br> &emsp; -> select 'New' in 'Name of the table or the view'
    	<br> &emsp; -> change table name to 'temp_account' and change data type if needed -> Ok
      	<br> &emsp; -> now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
      	<br> &emsp; -> now, in 'Control Flow' drag 'Execute SQL Task'
      	<br> &emsp; -> double click on it
  	<br> &emsp; -> in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_dw' in 'Connection'
       	<br> &emsp; -> add `update d
set	d.cust_name	= t.cust_name,
	d.cust_add = t.cust_add,
	d.cust_state = t.cust_state,
	d.cust_zipcode = t.cust_zipcode,
	d.br_id = t.br_id,
	d.prod_id = t.prod_id,
	d.prod_name = t.prod_name,
	d.status = t.status
from dim_account d join temp_account t on d.acc_id = t.acc_id
go
truncate table temp_account
go` in 'SQLStatement' -> Ok -> Ok

  <br> -> Incremental loading using 'Slowly Changing Dimension'
  <br> -> double click on 'DWH_Load_dim_branch'
  <br> -> drag 'Slowly Changing Dimension' just before 'OLE DB Destination' i.e., data loading
        <br> &emsp; -> connect 'blue pipe' from 'Lookup' to 'Slowly Changing Dimension'
      	<br> &emsp; -> double click on it
      	<br> &emsp; -> Next -> select '[dbo].[dim_branch]' in 'Table or view' -> choose the 'Business key' column
      	<br> &emsp; -> **Remember**
      	<br> &emsp; -> Fixed Attribute -> No update or change
      	<br> &emsp; -> Changing Attribute -> Overwriting the data
      	<br> &emsp; -> Historical Attribute -> maintain historic data
      	<br> &emsp; -> Next -> now select dimension column in 'Dimension Columns' and select 'Changing Attribute' in 'Change Type' -> Next -> Next -> Next -> Finish
      	<br> &emsp; -> just remember performance wise 'Slowly Changing Dimension' is not so good because it uses 'OLE DB Command' for data loading and as we know 'OLE DB Command' perform row by row wise.
  
  <br> -> Incremental loading using 'Stored Procedure'
  <br> -> load data from 'bank' server to 'bank_stage' server
  <br> -> for example we are going to load 'account_stage' table in 'bank_stage' server from 'account' table in 'bank' server
  <br> -> now open SQL Server Management Studio
  <br> -> creating a stored procedure in 'bank_stage' that will load data from 'account' to 'account_stage'
  ```sql
  create proc usp_load_account_stage
  as
  begin
  	--delete all rows from destination
  	truncate table account_stage
  	--load all data from source to destination
  	insert into account_stage
  	select * from bank.dbo.account_table
  end
  ```
  <br> -> calling the stored procedure `exec usp_load_account_stage`
  <br> -> now loading 'dim_account' in 'bank_dw' from 'account_stage' in 'bank_stage' and few columns will add and delete in final dimension table
  <br> -> also note that in data warehouse we will load data incrementally not full loading
  <br> -> creating a stored procedure that will load data incrementally from 'account_stage' to 'dim_account'
  ```sql
  create proc usp_incr_load_dim_account
  as
  begin
  	--Incremental Insert
  		--select columns that we need for dimension table
  	insert into dim_account
  	select	a.acc_id, a.cust_name, a.cust_add, a.cust_state, a.cust_zipcode, a.br_id,
  			a.prod_id, p.prod_name, a.status, 91 as 'country_code'
  		--joining two stage table to get 'prod_name' column
  	from bank_stage.dbo.account_stage a join bank_stage.dbo.product_stage p
  	on a.prod_id = p.prod_id
  		--left joining with 'dim_account' column to know about new data
  	left join dim_account d on a.acc_id = d.acc_id
  	where d.acc_id is null

  	--Incremental Update
  	update d
  	set d.cust_name = a.cust_name, d.cust_add = a.cust_add,
  		d.cust_state = a.cust_state, d.cust_zipcode = a.cust_zipcode, d.br_id = a.br_id,
  		d.prod_id = a.prod_id, d.prod_name = p.prod_name, d.status = a.status
  		--joining two stage table to get 'prod_name' column
  	from bank_stage.dbo.account_stage a join bank_stage.dbo.product_stage p
  	on a.prod_id = p.prod_id
  		--joining with 'dim_account' column to know about the updated data
  	join dim_account d on a.acc_id = d.acc_id
  	where d.cust_name != a.cust_name or d.cust_add != a.cust_add or
  		d.cust_state != a.cust_state or d.cust_zipcode != a.cust_zipcode or d.br_id != a.br_id or
  		d.prod_id != a.prod_id or d.prod_name != p.prod_name or d.status != a.status
  end
  ```
  <br> -> calling the stored procedure `exec usp_incr_load_dim_account`
  <br> -> major drawback in 'Stored Procedure' is to maintain the code and writing the complex code for more columns.

  <br> -> Incremental loading using 'Merge Statement'
  <br> -> A 'Merge Statement' is a SQL statement that performs INSERT, UPDATE, and DELETE operations based on the existence of rows matching the selection criteria in the target table.
  <br> -> now loading 'dim_account' in 'bank_dw' from 'account_stage' in 'bank_stage' and few columns will add and delete in final dimension table
  <br> -> creating a 'merge statement' that will load data incrementally from 'account_stage' to 'dim_account'
  ```sql
  create proc usp_merge_dim_account
  as
  begin

  	--inserting source table into temp table
  	select	a.acc_id, a.cust_name, a.cust_add, a.cust_state, a.cust_zipcode, a.br_id, a.prod_id,
  			p.prod_name, a.status, 91 as 'country_code' into #temp_source
  	from bank_stage.dbo.account_stage a join bank_stage.dbo.product_stage p
  	on a.prod_id = p.prod_id

  	merge dim_account as d -- destination table
  	using #temp_source as s -- source table
  	on s.acc_id= d.acc_id

  	-- insert
  	when not matched by target
  	then
  	insert(acc_id, cust_name, cust_add, cust_state, cust_zipcode, br_id, prod_id, prod_name, status, country_code)
  	values(s.acc_id, s.cust_name, s.cust_add, s.cust_state, s.cust_zipcode, s.br_id, s.prod_id, s.prod_name, s.status, s.country_code)

  	--update
  	when	matched and s.cust_name <> d.cust_name or s.cust_add <> d.cust_add or
  			s.cust_state <> d.cust_state or s.cust_zipcode <> d.cust_zipcode or
  			s.br_id <> d.br_id or s.prod_id <> d.prod_id or s.prod_name <> d.prod_name or
  			s.status <> d.status or s.country_code <> d.country_code
  	then
  	update
  	set	d.cust_name = s.cust_name, d.cust_add = s.cust_add, d.cust_state = s.cust_state,
  		d.cust_zipcode = s.cust_zipcode, d.br_id = s.br_id, d.prod_id = s.prod_id,
  		d.prod_name = s.prod_name, d.status = s.status, d.country_code = s.country_code;
  end
  ```

### SSIS Deployment
- DEV tasks
  - Create a SSIS Package / Perform Unit Testing
  - Implement Logging in SSIS
    - Extensions -> SSIS -> Logging -> tick the 'Containers' from left box -> click add 'SSIS log provider for SQL Server' and 'SSIS log provider for Windows Event Log' in 'Providers and Logs' tab -> tick previous two log below -> select a server for 'SSIS log provider for SQL Server' in 'Configuration' -> in 'Details' tab selects 'OnError' & 'OnPostExecute' & 'OnTaskFailed' -> OK
    - Event Handler (Customized Logging)
  - Build and CheckIn the Code to VSTF / TFS
    - right click on project name and click on 'Build' to find any issue with the project -> now go to project path 'D:\Books\MSBI\Project\ssis_bank_dw\bin\Development' there must be an .ispac file, this file having all the packages -> and upload this .ispac file to VSTF/TFS server -> now you have to give deployment guide as .txt or .docx file like
      <br> 'Project Name:
      <br> Author:
      <br> Purpose:
      <br> Steps:
      1. go to VSTF server and download <name> folder and its content
      2. go to DWH and open SQL Server
      3. right click on integration service catalogs and choose create a catalog
      4. right click on SSISDB and choose folder
      5. name = bank and ok
      6. double click on .ispac file
      7. Next
      8. choose 'SSIS in SQL Server' and Next
      9. select Server
      10. click on ‘Connect’
      11. Browse for the folder
      12. Next
      13. Deploy
      14. Close'
- DBA tasks
  - Download all files from VSTF/TFS server to DWH server
  - Create Integration Services Catalogs in SQL Server of DWH
    - open SSMS -> right click on 'Integration Service Catalogs' -> click on 'Create Catalog' -> click on 'Enable CLR Integration' -> enter password '12345' or anything you want -> OK -> now click on SSISDB and click on 'Create Folder' -> put name -> OK
  - Deploy the package using ispac file
    - double click on .ispac file -> next -> choose 'SSIS in SQL Server' and ‘Next’ -> put 'Server Name' as '.' -> click on 'Connect' -> now 'Browse' for the folder which was created in SSMS Integration Service Catalogs -> select the folder and change the name as you wish in 'Path' -> Next -> click on 'Deploy' -> Close
    - now go to SSMS and refresh 'Integration Service Catalogs' -> and all the packages will be there in the folder which was created earlier -> now create a document what the packages are doing
    - in real life all the servers will be different so in that case we need to configure the connections -> go to SSMS and open the folder in 'Integration Service Catalogs' -> right click on packages and click on 'Configure' -> go to 'Connection Manager' tab -> change server name -> click on 3 dots -> click on 'Edit values' and enter server address -> ok -> ok -> right click on packages and click on 'Execute' -> ok, this will execute the task -> click 'Yes' to report the task -> report will generate and also you can find the SQL Server logging in System Table in Server (Stage_company -> Tables -> System Tables)
  - Schedule a Package Using SQL Server Jobs
    - to schedule task -> go to 'SQL Server Agent' and start it by right -> expand it and right click on 'Jobs' -> click on 'New Job' -> put job name 'job_forloop' in 'General' tab -> click on 'Steps' tab -> click on 'New' -> put 'Step Name' -> select type 'SQL Server Integration Services Packages' -> now in below 'Package' tab choose 'SSIS Catalog' in 'Package source' -> put 'Server' -> now click on 3 dots in 'Package' and choose one package -> ok -> now go to 'Schedule' tab -> click on 'New' -> put 'Name' -> choose 'Occurs' as daily, weekly, monthly -> click on 'Occurs once at' and put time -> ok -> ok -> now task will automatically run at 01:00AM daily or you can run the same at any time -> right click on the task and click on 'Start job at step' -> and you can watch the logging, by double clicking on 'Job Activity Monitor'

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
  - new
  - change provider from 'Native OLE DB' to 'Microsoft OLE DB Driver for SQL Server'
  - enter 'Server or file name' or '.'
  - select or enter a database name in 'Initial catalog'
  - check 'Test Connection' - ok - ok
  - next
  - select the 'Use the service account'
  - next
  - enter data source name 'ds_bank_dw'
  - finish
  
- **Create Data Source View**
  - right click on 'Data Source Views'
  - select ‘New Data Source View’
  - next
  - select data source from 'Relational Data Sources'
  - next
  - select fact tables first
  - click on ‘Add Related Tables’ to select all other tables related to fact tables
  - next
  - enter name 'dsv_bank_dw'
  - finish
  - match the relationships if needed
  - basically, you need to create multiple view as per your requirements and on the basis of view you will create cubes

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
  - now enter 'Cube name:' as 'cube_bank_dw'
  - finish

- **Build Dimension**
  - Dimensions will automatically create when cube is formed
  
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
  - browser
  - reconnect
- click on 'Browser' (to see all the aggregations)
-- to view new items -> double click on dimensions in solution explorer -> drag items from ‘data source view’ to ‘attributes’ -> is hierarchy is needed then drag items from ‘attributes’ to ‘hierarchies’ -> enter name of the hierarchy -> save -> ok -> if we need, we can create more hierarchies
-- now again rebuild and process the cube if any changes are done
-- now go to browser and refresh or reconnect the cube
-- now you can use this cube
-- excel -> data -> from database -> from analysis services -> enter server name or . -> log on credential ‘use windows authentication’ -> next -> select cube -> next -> finish -> ok -> now do anything as your wish
