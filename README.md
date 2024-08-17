# CALL-CENTER-ANALYSIS_POWER-BI-REPORT

## Table of content 

## PROBLEM STATEMENT
 ServiceSpot, an IT company, contacted us to help them analyze their call center data. 
They receive daily calls from their customers and would like to know how things are going. The data is split across multiple files, and they are unable to make good use of it. 

Our  mission is to provide them with a complete solution which will help them analyze through nice looking reports.
## DATASET
The dataset folder contains 7 files. 
Columns description can be access here : 
[Call Center Analysis - Data Dictionnary]([../blob/master/LICENSE](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Data%20Dictionnary%20-%20Call%20Center%20Analysis.pdf))

| FILE NAME        | FILE DESCRIPTION           | COLUMNS NAME  |
| ------------- |:-------------| :-----:|
| **Data YYYY** (YYYY being the year-1 file per year from 2018 to 2021) (4 CSV files)| Record of the calls received during the year mentioning the data and time of occurrence and other main information | *CallTimestamp*, *Call Type*, *EmployeeID*, *CallDuration*,*WaitTime*, *CallAbandoned*. |
| **Lookup Data** (Excel file)   | Made of two tabs with one table each. The employee table allows to identify an employee by name and ID with his attributed call center site. |   *EmployeeID*, *EmployeeName*, *Site*, *ManagerName*|
| | The call Type Table describe the possible call categories i.e the call purpose.|*CallTypeID*, *CallTypeLabel* |
| **US States** (Excel file)| Identify all states in the USA       | *StateCD*, *Name*, *Region* |
| **Call Charges** 'Excel file) | The call charges fee according to the call type and the year       |   *Call Type Key*, *Call Type*, *Call Charges/min (2018)*, *Call Charges/min (2019)*, *Call Charges/min (2020)*, *Call Charges/min (2021)* |

## PROJECT WORKFLOW

  ### STEP 1 : Transforming and cleaning the data in Power Query
In the effort to minimize the number of tables in the model and avoid data redundancy, given the data in our possession we have decided that __the transformation should aim at creating 3 tables: Call Data, Employee and Site.__ 

The Calls table will consolidate call data from the past four years and be enriched with information from the call charges and call type tables. This approach ensures that no data is lost and eliminates the need to load both tables into the model. As the __Calls table will serve as our fact table__, it will also include a common column for __the dimension tables : Site and Employee__.

The Site table will contain the information previously included in the US State table therefore avoiding the load of the US State table in the model.

No additional information will be added to the Employee table
