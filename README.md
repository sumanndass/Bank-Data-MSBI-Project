# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)
- [SSIS for Stage Database (SQL Server Integration Service)](#ssis-for-stage-database-sql-server-integration-service)
- [SSIS for Data Warehouse Database (SQL Server Integration Service)](#ssis-for-data-warehouse-database-sql-server-integration-service)

### Overview
- This is a daily bank operation project.
- Assume daily few data are storing in 'bank' database and the tables are 'account_table' and 'transaction_table'.
- Also, few data are storing in 'product_doc' excel file.
- Also, few data are storing in 'branch_staff_doc' excel file and sheets are 'branch' and 'staff'.
- We need to load the data from 'bank' and 'bank_doc' to 'bank_stage' database and perform ETL operation on the same.

### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - 
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
<br> Create SSIS project using 'Integration Service Project' in DevEnv

- **Create SSIS Package for Stage Server**
<br> open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
<br> we have 2 database table and 2 excel file, one excel workbook has two sheets. So, we need to create 5 packages, named 'account_db', 'transaction_db', 'branch_doc', 'staff_doc', 'product_doc'.

	**Remember**
  <br> Data Flow - ETL Activities
	<br> Control Flow - Non-ETL Activities

- **Data Loading to 'bank_stage' Database from 'bank' Database**
  <br> -> double click on 'account_db' SSIS Packages
  <br> -> drag 'Data Flow Task' in 'Control Flow' section
  <br> -> double click on 'Data Flow Task'
  <br> -> drag 'OLE DB Source'
     	<br> &emsp; -> double click on it
  	<br> &emsp; -> in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
  	<br> &emsp; -> select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
  	<br> &emsp; -> put 'Server or file name' as '.' -> select database name 'bank' in 'Initial catalog' -> Ok -> Ok
  	<br> &emsp; -> select 'Data access mode' as 'Table or view'
  	<br> &emsp; -> choose '[dbo].[account_table]' from 'Name of the table or the view' -> Ok
  <br> -> drag 'OLE DB Destination'
   	<br> &emsp; -> connect 'blue pipe' from source to destination
       	<br> &emsp; -> double click on it
   	<br> &emsp; -> in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
  	<br> &emsp; -> select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
  	<br> &emsp; -> put 'Server or file name' as '.' -> select database name 'bank_stage' in 'Initial catalog' -> Ok -> Ok
  	<br> &emsp; -> select 'Data access mode' as 'Table or view - fast load'
  	<br> &emsp; -> select 'New' in 'Name of the table or the view'
    	<br> &emsp; -> change table name to 'account_stage' and change data type if needed -> Ok
      	<br> &emsp; -> now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
  <br> -> change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  <br> -> stage table always needs fresh data
  <br> -> so, drag 'Execute SQL Task' in 'Control Flow'
     	<br> &emsp; -> double click on it
       	<br> &emsp; -> in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
       	<br> &emsp; -> add SQL truncate command (`truncate table account_stage`) to delete all old data from stage whenever new data comes -> Ok
  	<br> &emsp; -> connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
       	<br> &emsp; -> 'Start' the project
  	<br> &emsp; -> do the same for 'transaction_db' package

- **Data Loading to 'bank_stage' Database from 'Excel' Document**
<br> -> double click on 'branch_doc' SSIS Packages
<br> -> drag 'Data Flow Task' in 'Control Flow' section
<br> -> double click on 'Data Flow Task'
<br> -> drag 'Excel Source'
     	<br> &emsp; -> double click on it
  	<br> &emsp; -> in 'connection Manager' select 'New'
  	<br> &emsp; -> find the 'excel' document path and choose the same and select proper 'Excel version' -> Ok
  	<br> &emsp; -> select 'Data access mode' as 'Table or view'
  	<br> &emsp; -> choose 'branch$' from 'Name of the Excel sheet' -> Ok
<br> -> drag 'Data Conversion'
     	<br> &emsp; -> connect 'blue pipe' from 'Excel Source' to 'Data Conversion'
     	<br> &emsp; -> double click on it
     	<br> &emsp; -> select the 'Available Input Columns'
     	<br> &emsp; -> change the 'Data Type' and 'Length' -> Ok
<br> -> drag 'OLE DB Destination'
   	<br> &emsp; -> connect 'blue pipe' from 'Data Conversion' to 'OLE DB Destination'
       	<br> &emsp; -> double click on it
   	<br> &emsp; -> in 'connection Manager' select 'dest.bank_stage' (which was saved earlier) in 'OLE DB Connection manager'
  	<br> &emsp; -> select 'Data access mode' as 'Table or view - fast load'
  	<br> &emsp; -> select 'New' in 'Name of the table or the view'
    	<br> &emsp; -> change table name to 'branch_stage' and change data type if needed -> Ok
      	<br> &emsp; -> now click on 'Mappings' choose proper 'Input Column' and 'Destination Column' -> Ok
<br> -> change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
<br> -> stage table always needs fresh data
<br> -> so, drag 'Execute SQL Task' in 'Control Flow'
     	<br> &emsp; -> double click on it
       	<br> &emsp; -> in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
       	<br> &emsp; -> add SQL truncate command (`truncate table account_stage`) to delete all old data from stage whenever new data comes -> Ok
  	<br> &emsp; -> connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
       	<br> &emsp; -> 'Start' the project
  	<br> &emsp; -> do the same for 'staff_doc' and 'product_doc' packages

- **SSIS Logging (To know about the status of the successful loadings)**
  <br> create a SSIS logging table named 'ssis_log' (where SSIS status will store)
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
  <br> now we will store SSIS loading status in newly created table
  <br> -> open 'account_db' package and drag 'Execute SQL Task' in 'Control Flow'
  <br> -> double click on it
  <br> -> in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
  <br> -> add 'SQLStatement' command (`insert into ssis_log values(?, getdate(), ?, 'Success...')`)
  <br> -> in 'Parameter Mapping' click on 'Add' and choose 'System::PackageName' in 'Variable Name' for first '?' means 0th position
  <br> -> choose 'Varchar' as 'Data type', choose '0' in 'Parameter Name' for the 0th position '?', choose '-1' for 'Parameter Size' -> Ok
  <br> -> now click on 'Variables' and select 'Add variable' -> choose 'Name' like: 'row_cnt'
  <br> -> now in 'Data Flow', drag 'Row Count'
  <br> -> connect 'blue pipe' from 'OLE DB Source' to 'Row Count'
  <br> -> double click on 'Row Count' and select the 'User::row_cnt' variable
  <br> -> connect 'blue pipe' from 'Row Count' to 'OLE DB Destination'
  <br> -> now in 'Control Flow', double click on newly created 'Execute SQL Task'
  <br> -> in 'Parameter Mapping' click on 'Add' and choose 'User::row_cnt' in 'Variable Name' for second '?' means 1st position
  <br> -> choose 'Large_integer' as 'Data type', choose '1' in 'Parameter Name' for the 1st position '?', choose '-1' for 'Parameter Size' -> Ok
  <br> -> connect 'green pipe' from 'Data Flow Task' to new 'Execute SQL Task 1'
  <br> -> 'Start' the project
  <br> -> do the same for 'transaction_db', 'branch_doc', 'staff_doc', 'product_doc' packages

- **SSIS Failure Logging (To know about the failure happened in loading)**
  <br> -> double click on desired package -> go to 'Extension' -> 'SSIS' -> 'Logging'
  <br> -> choose all 'Containers:' -> 'Add' 'SSIS log provider for SQL Server', tick the same and choose the destination server 'bank_stage' in 'Configuration' -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  <br> -> Find the error logs in system tables in the selected database by using command `sql select * from sysssislog`
  <br> -> again, choose all 'Containers:' -> add 'SSIS log provider for Windows Event Log', tick the same -> go to 'Details' and tick 'OnError' and 'OnTaskFailed'
  <br> -> Find the error logs in 'Windows Event Viewer' -> 'Windows Logs' -> 'Application'
  <br> -> Set this task for other packages also

### SSIS for Data Warehouse Database (SQL Server Integration Service)
- **Create Data Warehouse Database**
  ```sql
  create database bank_dw
  go

  use bank_dw
  go
  ```

- **Create Dimension and Fact table in DWH**
  <br> As in OLAP/DWH we need to denormalized dimension tables and extract facts, so
  <br> we merged account_table and product_table
  <br> we merged branch_table and region_table
  <br> we merged transaction_table and staff_table
  <br> we created a new dimension date table
  <br> we created a new dimension location table
  <br> we extracted facts from account table and created a new fact table
  <br> we extracted facts from transaction table and created a new fact table

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
  	br_state	varchar(30) 	foreign key references dim_location(state),
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
  	cust_state		varchar(30) 	foreign key references dim_location(state),
  	cust_zipcode		varchar(6),
  	br_id			varchar(4)	foreign key references dim_branch(br_id),
  	prod_id			varchar(2),
  	prod_name		varchar(20),
  	status			varchar(1)
  )
  ```
  ```sql
  create table dim_transaction
  (
  	tran_id		int		primary key,
  	acc_id		int		foreign key references dim_account(acc_id),
  	br_id		varchar(4)	foreign key references dim_branch(br_id),
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
  	acc_id			int		foreign key references dim_account(acc_id),
  	doo_date_id		int		foreign key references dim_date(date_id),
  	clr_bal			money,
  	unclr_bal		money
  )
  ```
  ```sql
  create table fact_transaction
  (
  	tran_id			int		foreign key references dim_transaction(tran_id),
  	dot_date_id		int		foreign key references dim_date(date_id),
  	txn_amt			money
  )
  ```

- **Create Reference Tables/Views in DWH**
  <br> As there have so many changes in tables in DWH so, we need to have reference table/views, from where we will get the data.
  <br> We will use referencee table becuase we will use more options in SSIS. By the way, below reference views also can be used for data fetching from bank_stage to bank_dw.
 
  <br> reference view because account_table and product_table merged
  ```sql
  create view vw_ref_dim_account
  as
  select a.acc_id, a.cust_name, a.cust_add, a.cust_state,
  a.cust_zipcode, a.br_id, a.prod_id, p.prod_name, a.status
  from bank.dbo.account_table a join bank.dbo.product_table p on a.prod_id = p.prod_id
  ```

  <br> reference view because branch_table and region_table merged
  ```sql
  create view vw_ref_dim_branch
  as
  select b.br_id, b.br_name, b.br_add, b.br_state, b.br_zipcode, r.reg_id, r.reg_name
  from bank.dbo.branch_table b join bank.dbo.region_table r on b.reg_id = r.reg_id
  ```

  <br> reference view because transaction_table and staff_table merged
  ```sql
  create view vw_ref_dim_transaction
  as
  select t.tran_id, t.acc_id, t.br_id, t.txn_type, t.chq_no, t.chq_date, t.staff_id,
  s.staff_name, s.designation
  from bank.dbo.transaction_table t join bank.dbo.staff_table s on t.staff_id = s.staff_id
  ```

  <br> populate new date dimension table dim_date
  ```sql
  declare @startdate datetime
  declare @enddate datetime = getdate()

  select @startdate = min(doo) from bank.dbo.account_table

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
  
  <br> populate new dimension location table dim_location
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

  <br> we have 7 dimension tables and 2 fact tables. So, we need to create 7 packages, named 'DWH_Load_dim_branch', 'DWH_Load_dim_account', 'DWH_Load_dim_transsaction', 'DWH_Load_fact_account', 'DWH_Load_fact_transaction' and 2 derived dimension tables ('dim_date' and 'dim_location') are already loaded.

- **Create SSIS Package for DWH Server**
<br> open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages

- **Data Loading to 'bank_dw' Database from 'bank_stage' Database**
  <br> -> double click on 'DWH_Load_dim_account' SSIS Packages
  <br> -> drag 'Data Flow Task' in 'Control Flow' section
  <br> -> double click on 'Data Flow Task'
  <br> -> drag 'OLE DB Source'
     	<br> &emsp; -> double click on it
  	<br> &emsp; -> in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
  	<br> &emsp; -> select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
  	<br> &emsp; -> put 'Server or file name' as '.' -> select database name 'bank_stage' in 'Initial catalog' -> Ok -> Ok
  	<br> &emsp; -> select 'Data access mode' as 'Table or view'
  	<br> &emsp; -> choose '[dbo].[account_stage]' from 'Name of the table or the view' -> Ok
  <br> -> drag 'Lookup'
    	<br> &emsp; -> Look up on 'product_stage' table based on 'prod_id' and get 'prod_name', reference 'ETL_Mapping _Doc.xlsx'
      	<br> &emsp; -> connect 'blue pipe' from 'OLE DB Source' to 'Lookup'
      	<br> &emsp; -> double click on it
      	<br> &emsp; -> choose 'Redirect rows to no match output' in '﻿Specify how to handle rows with no matching entries' in 'General'
	<br> &emsp; -> in 'Connection' choose 'bank_stage' in 'OLE DB Connection Manager' and choose 'product_stage' in 'Use a table or a view'
      	<br> &emsp; -> in 'Columns' drag 'prod_id' of 'Available Input Columns' on 'prod_id' of 'Available Input Columns' and tick desired column 'prod_name' -> Ok
  <br> -> drag 'OLE DB Destination'
   	<br> &emsp; -> connect 'blue pipe' from 'Lookup' to 'OLE DB Destination'
       	<br> &emsp; -> double click on it
   	<br> &emsp; -> in 'connection Manager' select 'New' in 'OLE DB Connection manager' -> againg select 'New' -> Ok
  	<br> &emsp; -> select 'Provider' as 'Native OLE DB\Microsoft OLE DB Driver for SQL Server'
  	<br> &emsp; -> put 'Server or file name' as '.' -> select database name 'bank_dw' in 'Initial catalog' -> Ok -> Ok
  	<br> &emsp; -> select 'Data access mode' as 'Table or view - fast load'
  	<br> &emsp; -> select 'New' in 'Name of the table or the view'
    	<br> &emsp; -> change table name to 'dim_account' and change data type if needed -> Ok
      	<br> &emsp; -> now click on 'Mappings' to check source and destination column and data type are corrected or not -> Ok
  <br> -> change names in 'Connection Managers' for better understanding -> right click on it and 'Convert to Package Connection' for rest of the project
  <br> -> stage table always needs fresh data
  <br> -> so, drag 'Execute SQL Task' in 'Control Flow'
     	<br> &emsp; -> double click on it
       	<br> &emsp; -> in 'General' select 'OLE DB' in 'ConnectionType' and 'bank_stage' in 'Connection'
       	<br> &emsp; -> add SQL truncate command (`truncate table account_stage`) to delete all old data from stage whenever new data comes -> Ok
  	<br> &emsp; -> connect 'green pipe' from 'Execute SQL task' to 'Data Flow Task'
       	<br> &emsp; -> 'Start' the project
  	<br> &emsp; -> do the same for 'transaction_db' package
