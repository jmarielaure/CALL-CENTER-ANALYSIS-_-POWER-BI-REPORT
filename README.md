# CALL-CENTER-ANALYSIS_POWER-BI-REPORT



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
| **Call Charges** (Excel file) | The call charges fee according to the call type and the year       |   *Call Type Key*, *Call Type*, *Call Charges/min (2018)*, *Call Charges/min (2019)*, *Call Charges/min (2020)*, *Call Charges/min (2021)* |

## PROJECT WORKFLOW

  ## STEP 1 : TRANSFORMING AND CLEANING DATA WITH POWER QUERY
In the effort to minimize the number of tables in the model and avoid data redundancy, given the data in our possession we have decided that __the transformation should aim at creating 3 tables: Call Data, Employee and Site.__ 

The Calls table will consolidate call data from the past four years and be enriched with information from the call charges and call type tables. This approach ensures that no data is lost and eliminates the need to load both tables into the model. As the __Calls table will serve as our fact table__, it will also include a common column for __the dimension tables : Site and Employee__.

The Site table will contain the information previously included in the US State table therefore avoiding the load of the US State table in the model.

No additional information will be added to the Employee table



   ### ■ __CALL DATA TABLE (fact table)__


#### Call Charges Table Transformation

- **Remove Null Rows**: Filter out any rows that contain null values to ensure data integrity.
- **Promote Headers**: Convert the first row of data into column headers for clarity.
- **Assign Datatypes**: Set appropriate data types for each column to facilitate accurate data processing.
- **Unpivot Columns**: Transform wide-formatted to obtain date information in a column instead of a row, making the data more suitable for analysis.
- **Correct Datatypes**: Recheck and correct any incorrect datatypes to ensure consistency.
- **Assign Local Currency**: Set the column with fees to use the US dollar currency format.
- **Extract Year**: Eextracting the year from the original pivoted column names, enhancing the temporal analysis capabilities.

#### Data YYYY Table Transformation

- **Split Columns**: Use a comma delimiter to split columns, separating data into distinct fields.
- **Separate Date and Time**: Extract and separate date and time into two different columns to improve query performance in Power BI.
- **Consolidation_Append Data Files**: Append the 4 transformed files into a new consolidated table named "Call Data".
- **Enrich Call Data Table**: Additional enhancements and enrichment steps display below were applied to the "Call Data" table to ensure comprehensive analysis.

| Type of Join  | Table                          | Column(s) Used as Join                            | Column(s) Retrieved |
|---------------|--------------------------------|---------------------------------------------------|---------------------|
| Right Join    | Call Data – Call Type          | CallTypeKey (called CallTypeID in the Call Type table) | CallTypeLabel       |
| Right Join    | Call Data – (Call) Charges     | Year YYYY and CallTypeLabel                       | Call Charges        |
| Right Join    | Call Data – Employee           | EmployeeID                                        | Sites               |

**Note:** the loading of the following tables was disabled : All 4 individual files Data YYYY, (Call) Charges and CallType.



   ### ■ __SITE TABLE (dimension table)__


#### Employee Table Transformation

- **Select columnn**: Keep only the Site column from the Employee table.
- **Remove duplicates**: Remove duplicates from Site column
- **Split column**: Split column by delimiter into twe separate column to aller the merge with the US State table : Site name and the stateCD.


  | Type of Join | Table          | Column(s) Used as Join | Column(s) Retrieved     |
  |--------------|----------------|------------------------|-------------------------|
  | Right Join   | Site – US State| StateCD                | City Name and Region     |

 **Note:** The loading of the US States table was disabled.

For all data loaded:
     The first row was promoted as the column name.
     Datatypes were assigned.
     Some columns were also renamed.
     Date and Time were separated in all tables with timestamps.
     Occasionally, columns were trimmed as invisible blank spaces were preventing perfect merging.


   ## STEP 2 : DATA MODELING

![Report Model View](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Report_Model_view.png "Model View")

To finish the model a calculated table “Calendar” was created directly in Power BI using DAX



```dax
Calendar =
ADDCOLUMNS (
CALENDAR (DATE(2018, 1, 1), DATE(2021, 12, 31)),
"Year", YEAR([Date]),
"Month", FORMAT([Date], "MMMM"),
"Month Number", MONTH([Date]),
"Quarter", "Q" & FORMAT(QUARTER([Date]), "0"),
"Day of Week", FORMAT([Date], "dddd"),
"Day of Year", FORMAT([Date], "DDD")
)
```

   ## STEP 3 : KPI DEFINITION AND DAX CALCULATION 

To see if the moment of the day had any significant impact on the number of calls received, we had to be able to group the calls into time brackets  and /or categories (morning , afternoon, night) according to the time.
We created calculated columns Time_Brackets and Time_Period assigning a time bracket and a period of the day to each call according to the time of call. 
A hierarchy was also created : Time Period  Time_Brackets

-	__Time_brackets DAX calculation__
```dax
Time_Bracket = 
VAR start_hour =
    FORMAT ( Data[Call  Time], "hh" )
VAR end_hour =
    IF ( LEN ( start_hour + 1 ) = 1, "0" & start_hour + 1, start_hour + 1 )
RETURN
    start_hour & ":00 - " & end_hour & ":00"

```
  
-	__Time_Period DAX calculation__
  ```dax
Time_Period = 
VAR start_hour =
    VALUE(FORMAT(Data[Call  Time], "hh"))
RETURN
    SWITCH (
        TRUE(),
        start_hour < 12, "Morning",
        start_hour >= 12 && start_hour < 18, "Afternoon",
        "Night"
    )
```

   ## STEP 4 : REPORT CREATION WITH POWER BI

To see if the moment of the day had any significant impact on the number of calls received, we had to be able to group the calls into time brackets  and /or categories (morning , afternoon, night) according to the time.
We created calculated 

   ### ■ __OVERVIEW PAGE__

Focus on primary information related to the calls. This page answer basic questions such as : which site handles the most calls, what are the customers calling for, when does the customers mostly call , etc.

![Report Overview tab](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Report_Overview%20tab.png "Report Overview Tab")

- __Filter pane features__
  
Allow you to filter all visuals on the page. The period filter is a drop-down list with a time hierarchy ( year, quarter, month). The button “ Clear All Slicers” was added for time saving purposes.

![Overview filter pane](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Overview%20page%20_%20Filter%20Pane.png "Overview filter pane")

- __Cards with KPI and page navigation button at the top row__
  
The top row was used to display the main KPIs related to the calls : total calls, rate of call abandonment, revenue generated, average wait time in second and average call duration in min. 

![Overview tab top row](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Overview%20page%20_%20KPI%20Cards.png "Overview teb top row")

- __Double axis line chart__
Line chart to put in perspective the waiting time and the  call abandoned rate based on the time of the day

![Line chart](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Overview%20page%20_%20Double%20axis%20Line%20chart%20_%20call%20abandoned%20vs%20wait%20time.png "Double axis Line chart trend comparison between average wait time and call abandonment rate")

- __Pie chart and stacked column__
A pie chart shows the total call repartition among the different sites.
We can drill down the stacked column visuals to have the call type repartition on a specific period.

![Pie chart and stacked column](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Overview%20page%20_%20Bottom%20row.png "Pie chart and stacked column with drill down option")



   ### ■ __PERFORMANCE PAGE – FINANCIAL VIEW__
 Focus on information related to revenue. It can be accessed thanks to the button “ Financial Performance“ which activates the dedicated bookmark. 

![PERFORMANCE PAGE – FINANCIAL VIEW](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Report_%20Performance%20Insight%20tab_%20Financial%20Performance%20view.png "PERFORMANCE PAGE – FINANCIAL VIEW")




   ### ■ __PERFORMANCE PAGE – EMPLOYEE PERFORMANCE VIEW__
   
Focus on information related to employee’s performance. It can be accessed thanks to the button “ Employee Performance“ which activates the dedicated bookmark. 
We can answer questions such as which employees handle the most calls on each site, which employee generate the most revenue, etc.

![EMPLOYEE PERFORMANCE VIEW](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Report%20screenshots/Report_%20Performance%20Insight%20tab_%20Employee%20Performance%20view.png "EMPLOYEE PERFORMANCE VIEW")


## KEY FINDINGS AND RECOMMANDATION (extract)


- __Key metrics__



| Metric                  | Value                                    |
|-------------------------|------------------------------------------|
| **Total Calls**         | 132K over the 4-year evaluated period    |
| **Call Abandonment**    | 7,923 calls abandoned (6.01%)            |
| **Average Wait Time**   | 29.7 seconds                             |
| **Average Call Duration** | 13 minutes                             |



- __Financial Performance__


| Metric                       | Value                                          |
|------------------------------|------------------------------------------------|
| **Average Charge per Call**  | $15.94                                         |
| **Total Call Charges**       | $2M, indicating robust revenue from call operations |
| **Average Calls per Day**    | 90 calls/day                                   |


  
- __Main recommandation__

__Improve the usability of their product:__ Each year, more than 50% of the calls are for Tech Support (approx. 16k calls/year). Servicespot should focus on improving the ease of use of their products and services to reduce this type of call. It might be wise to review and improve the product, as well as to give customers clear indication and enhance user training to reduce dependency on technical support. 


_For further insight , here is the link to the
[Insights and recommandations PDF report]([../blob/master/LICENSE](https://github.com/jmarielaure/CALL-CENTER-ANALYSIS-_-POWER-BI-REPORT/blob/main/Insight%20and%20Recommandations%20Report%20_%20Call%20center%20analysis.pdf))_

## TOOLS USED
PowerBI desktop, Power Query,  DAX

## CONTRIBUTION
Team project : Nada NAFIE and Sajeevan SOTHIRASA

## CONCLUSION

This project demonstrates the ability to extract business insights from a raw dataset going throught the different pahases of:

- __Data cleaning and transformation__

- __Modeling__: Creating a model in PowerBI with the use of Power Query

- __Optimize Data Processing__: We implemented data transformation strategies to streamline data handling, reducing redundancy and improving model efficiency.

- __Identify Key Metrics__: We established crucial performance indicators such as call abandonment rates, average wait times, and call charges.

- __Visualize Data Effectively__: Through Power BI, we created a comprehensive and interactive report that highlights trends, peak call times, and site performance.



__AREAS OF IMPROVEMENT/ COLLABORATION__
  
Future improvements could include incorporating more data, such as call resolutions and customer feedback and employee objectoves, to provide deeper insights.
Having data with more variabilty from one year to another would also led to a more interesting analysis.

To finish, by using automated ETL (Extract, Transform, Load) processes we would further reduce manual data preparation efforts and improve data accuracy. 
This same dataset was used in an ETL project showcasing the use of Miscrosoft SSIS. For more info, you can find the project repository at https://github.com/jmarielaure/ETL-Call-Center-Project


