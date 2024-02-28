# Bank-Data-MSBI-Project
## Table of Content
- [Overview](#overview)
- [Tools Used](#tools-used)

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
