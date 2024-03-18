# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)
- [SSIS for Stage Database (SQL Server Integration Service)](#ssis-for-stage-database-sql-server-integration-service)
- [SSIS for Data Warehouse Database (SQL Server Integration Service)](#ssis-for-data-warehouse-database-sql-server-integration-service)

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

### SSIS for Stage Database (SQL Server Integration Service)
- create a stage database named ‘bank_stage’ (where ETL will happen)
  ```sql
  create database bank_stage
  ```

- use the 'bank_stage' database
  ```sql
  use bank_stage
  ```

- Create SSIS Project
<br> Create SSIS project using ‘Integration Service Project’ in DevEnv

- Create SSIS Package for Stage Server
<br> open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
<br> we have 2 database table and 2 excel file, one excel workbook has two sheets. So, we need to create 5 packages, named `account_db`, `transaction_db`, `branch_doc`, `staff_doc`, `product_doc`

	**Remember**
  <br> Data Flow - ETL Activities
	<br> Control Flow - Non-ETL Activities

- Data Loading to `bank_stage` Server from `bank` Database
  <br> -> double click on SSIS Packages
  <br> -> drag `Data Flow Task` in `Control Flow` section
  <br> -> double click on `Data Flow Task`
  <br> -> drag `OLE DB Source` and double click on it
      &nbsp -> add source connection
  -> drag ‘OLE DB Destination’ and double click on it -> add destination
<br> -> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
<br> -> ‘Start’ the project

- Data Loading to Stage Server from Excel Document
<br> -> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘Excel Source’ and double click on it -> add source connection -> if anything mismatch in data type then drag  drag ‘OLE DB Destination’ and double click on it -> add destination
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

### SSIS for Data Warehouse Database (SQL Server Integration Service)
- Create Data Warehouse Database
  ```sql
  create database bank_dw
  go

  use bank_dw
  go
  ```

- Create Dimension and Fact table in DWH
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

- Create Reference Tables/Views in DWH
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

  <br> we have 7 dimension tables and 2 fact tables. So, we need to create 7 packages, named `DWH_Load_dim_branch`, `DWH_Load_dim_account`, `DWH_Load_dim_transsaction`, `DWH_Load_fact_account`, `DWH_Load_fact_transaction` and 2 derived dimension tables (`dim_date` and `dim_location`) are already loaded.

- Create SSIS Package for DWH Server
  <br> open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages

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
