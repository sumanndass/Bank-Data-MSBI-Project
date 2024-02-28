# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)
- [Database Creation](#database-creation)
- [SSIS (SQL Server Integration Service)](#SSIS-(SQL Server Integration Service))

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

- Create SSIS Package
open SSIS project -> in Solution Explorer -> create SSIS Packages -> rename the packages
we have 2 database table and 2 excel file, one excel workbook has two sheets. So, we need to create 5 packages, named account_db, transaction_db, branch_doc, staff_doc, product_doc

**Remember**
Data Flow - ETL Activities
Control Flow - Non-ETL Activities

- Data Loading from Database
-> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘OLE DB Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
-> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
-> ‘Start’ the project

- Data Loading from Excel
-> double click on SSIS Packages -> drag ‘Data Flow Task’ in ‘Control Flow’ section -> double click on ‘Data Flow Task' -> drag ‘Excel Source’ and double click on it -> add source connection -> drag ‘OLE DB Destination’ and double click on it -> add destination
-> change names in ‘Connection Manager’ for better understanding -> right click on it and ‘Convert to Package Connection’ for rest of the project
-> stage table always needs fresh data ->so, drag ‘Execute SQL Task’ in ‘Control Flow’ and double click on it -> add SQL truncate command to delete all old data from stage whenever new data comes
-> ‘Start’ the project

- SSIS Logging (To know about the status of the successful loadings)
  create a SSIS logging table named ‘ssis_log’ (where SSIS status will store)
   ```sql
   create table ssis_log
(
	id					int				primary key identity(1, 1),
	pkg_name			varchar(100)	not null,
	pkg_exec_time		datetime		not null,
	row_cnt				int				not null,
	pkg_exec_status		varchar(100)	not null
)
   ```
