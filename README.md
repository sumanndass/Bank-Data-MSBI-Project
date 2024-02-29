# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)
- [Database Creation](#database-creation)
- [SSIS (SQL Server Integration Service)](#ssis-sql-server-integration-service)

### Overview
- This is a daily bank operation project.
- Assume daily few data are storing in ‘bank’ database and the tables are ‘account_table’ and ‘transaction_table’.
- Also, few data are storing in ‘product_doc’ excel file.
- Also, few data are storing in ‘branch_staff_doc’ excel file and sheets are ‘branch’ and ‘staff’.
- We need to load the data from ‘bank’ and ‘bank_doc’ to ‘bank_stage’ database and perform ETL operation on the same.

### Tools Used
- SQL Server - Data Analysis, Data Cleaning
- SSIS - Extract, Transform, Loading
- SSAS - 
- SSRS -

### Database Creation
- create a stage database named ‘bank_stage’ (where ETL will happen)
  ```sql
  create database bank_stage
  ```
- use the 'bank_stage' database
  ```sql
  use bank_stage
  ```
### SSIS (SQL Server Integration Service)
- Create SSIS Project
Create SSIS project using ‘Integration Service Project’ in DevEnv

- Create SSIS Package for Stage Server
open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
we have 2 database table and 2 excel file, one excel workbook has two sheets. So, we need to create 5 packages, named account_db, transaction_db, branch_doc, staff_doc, product_doc

	**Remember**
  <br> Data Flow - ETL Activities
	<br> Control Flow - Non-ETL Activities

- Data Loading to Stage Server from Database
<br> -> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘OLE DB Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
<br> -> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
<br> -> ‘Start’ the project

- Data Loading to Stage Server from Excel
<br> -> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘Excel Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
<br> -> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
<br> -> ‘Start’ the project

- SSIS Logging (To know about the status of the successful loadings)
  <br> create a SSIS logging table named ‘ssis_log’ (where SSIS status will store)
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
  <br> now store SSIS loading status in newly created table
  <br> -> drag 'Execute SQL Task' in 'Control Flow' double click on it -> insert SQL Command ```sql insert into ssis_log values(?, getdate(), ?, 'success')``` -> in 'Parameter Mapping' use 'System::PackageName' for first '?' -> now in 'Data Flow', create a variable called 'row_cnt' -> drag 'Row Count', double click on it and select the 'row_cnt' variable -> now in 'Control Flow', double click on newly created 'Execute SQL Task' and in 'Parameter Mapping' use 'User::row_cnt' for second '?'
  <br> -> set this ‘Execute SQL Task’ for other packages also

- SSIS Failure Logging (To know about the failure happened in loading)
  <br> -> double click on desired package -> go to ‘Extension’ -> ‘SSIS’ -> ‘Logging’
-> choose all containers -> add ‘SSIS log provider for SQL Server’, tick the same and choose the destination server -> go to ‘Details’ and tick ‘OnError’ and ‘OnTaskFailed’
  <br> -> Find the error logs in system tables in the selected database by using command ```sql select * from sysssislog```
  <br> -> again, choose all containers -> add ‘SSIS log provider for Windows Event Log’, tick the same -> go to ‘Details’ and tick ‘OnError’ and ‘OnTaskFailed’
  <br> -> Find the error logs in Windows Event Viewer -> Windows Logs -> Application
  <br> -> Set this task for other packages also

- Create Data Warehouse Database
  ```sql
  create database bank_dw
  go

  use bank_dw
  go
  ```

- Create Dimension and Fact table in DWH
  <br> A fully denormalized database best for OLAP/DWH
  ```sql
  create table dim_date
  (
  	date_id		int			primary key,
  	full_date	datetime,
  	year_no		int,
  	sem_no		tinyint,
  	qtr_no		tinyint,
  	month_no	tinyint,
  	month_name	varchar(10),
  	week_no		tinyint,
  	day_no		tinyint,
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

- Create SSIS Package for DWH Server
open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages

- Data Loading to DWH Server from Stage Database
<br> -> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘OLE DB Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
<br> -> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
<br> -> ‘Start’ the project

- Data Loading to Stage Server from Excel
<br> -> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘Excel Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
<br> -> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
<br> -> ‘Start’ the project
