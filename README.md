# Lab 2: Advanced KQL, Policies, and Visualization

This Lab is organised into the following 4 challenges:
|Challenge | Description | Est. Time|
|--|--|--|
| [Challenge 5](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-5-how-long-will-my-data-be-kept---caching-and-retention-policies)| Caching and retention policies| 30 Min| 
| [Challenge 6](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-6-control-commands)| Control commands| 30 Min|
| [Challenge 7](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-7-going-more-advanced-with-kql)| Advanced KQL operators| 45 Min|
| [Challenge 8](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-8-visualization)| Visualization| 45 Min |


---
In order to receive the "ADX In a Day" digital badge, you will need to complete the challenges marked with 🎓. **Please submit the KQL queries/commands of these challenges in the following link**: [Answer sheet - ADX in a Day Lab 2](https://forms.office.com/r/J38XTAx09Q)

#### Please allow us 10 working days to issue the badge

<p align="center"><img src="/assets/images/badge.png" width="200"></p>

---
---
# [Go to ADX-In-A-Day Homepage](https://github.com/Azure/ADX-in-a-Day)

---
### Challenge 5: How long will my data be kept? - Caching and Retention Policies

Among the different policies you can set to the ADX cluster, two policies are of particular importance: 
- Retention policy (retention period)
- Cache policy (cache period)
First, a policy is used to enforce and control the properties of the cluster (or the database/table).
  
The **retention** policy: the time span, in days, for which it’s guaranteed that the data is kept available for querying. The time span is measured from the time that the records are ingested. When the period expires, the records  will not be available for querying any more. <br>
In other words, the retention policy defines the period during which data is retained and available to query, measured since ingestion time. Note that a large retention period may impact the cost. 

The **cache** policy: the time span, in days, for which to keep recently ingested data (which is usually the frequently queried data) available in the hot cache rather than in long term storage (this is also known as cold tier. Specifically, it is Azure blob storage). Data stored in the hot cache is actually stored in local SSD or the RAM of the machine, very close to the compute nodes. <br>
Therefore, more readily available for querying. The availability of data in the hot cache improves query performance but can potentially increase the cluster cost (as more data is being stored, more VMs are required to store it). In other words, the caching policy defines the period during which data is kept in the hot cache. 

All the data is always persisted in the cold tier, for the duration defined in the retention policy. Any data whose age falls within the hot cache policy will also be stored in the hot cache. If you query data from cold cache, it’s recommended to target a small specific range in time (“point in time”) for the queries to be efficient.

---
#### Challenge 5, Task 1: Change the retention policy via commands 🎓

Database policies can be overridden per table using a KQL control command.
ADX cluster and database are Azure resources. A database is a sub-resource of the cluster, so it can be edited from the portal. Tables are not considered an Azure resource, so they cannot be managed in the portal but via a KQL command.    
You can always use KQL commands to alter the policies of the entire Cluster/Database/tables. Table level policy takes precedence over database level which takes precedence over cluster level.

Alter the retention policy of the table ingestionLogs to 180 days.

[.alter table retention policy command](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/alter-table-retention-policy-command)

---

### Challenge 6: Control commands

#### Challenge 6, Task 1: .show/diagnostic logs/Insights
Control commands are requests to the service to retrieve information that is not necessarily data in the database tables, or to modify the service state, etc. In addition, they can be used to manage Azure Data Explorer.
The first character of the KQL text determines if the request is a control command or a query. Control commands must start with the dot (.) character, and no query may start with that character. <br><br>
[Management (control commands) overview](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/)

- The _‘.show queries’_ command returns a list of queries that have reached a final state, and that the user invoking the command has access to see.
- The _‘.show commands'_ command returns a table of the admin commands that have reached a final state.  The TotalCpu column is the value of the total CPU clock time (User mode + Kernel mode) consumed by this command.
- The _'.show journal'_ command returns a table that contains information about metadata operations that are done on the Azure Data Explorer database. The metadata operations can result from a control command that a user executed, or internal control commands that the system executed, such as drop extents by retention
- The '.show tables details' command returns  a set that contains the specified table or all tables in the database with a detailed summary of each table's properties.

---
#### Challenge 6, Task 2: Use .show queries 🎓

As part of an incident investigation, you need to find out how many queries were executed in the past 3 hours.
<br>
Write a command to count the number of queries that you ran (use the _User_ column), in the past 3 hours.

Reference:
[.show queries](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/queries)

---
#### Challenge 6, Task 3: Use .journal commands 🎓

Write a command to show the details of the function that you created earlier. When did you create the function? <br>
Hint: use the 'Event' and the 'EventTimestamp' columns.

Reference:
[.show journal](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/journal)

---
#### Challenge 6, Task 4: Use .show commands 🎓

Write a command to count the number commands that you run (use the User column), in the past 4 hours.

Reference:
[.show commands](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/commands)


---
#### Challenge 6, Task 5: Table details and size 🎓

Write a control command to show following details on all tables in the database.
- Use calculated columns using 'extend' operator to show the [extent](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/extents-overview) size of the data, per table size in GB. <br>

Hint: ADX provides function to format bytes into MB or GB or TB: format_bytes(*YourColumn*, 2) <br>

Reference:
- [.show table details](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/show-table-details-command)
- [format_bytes()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/format-bytesfunction)

---

### Challenge 7: Going more advanced with KQL

** Use ingestionLogs table for the following tasks.

#### Challenge 7, Task 1: Declaring variables and using 'let' statements 🎓

You can use the  **'let'** statement to set a variable name equal to an expression or a function, or to create views (virtual, temporary, tables based on the result-set of another KQL query). <br>

let statements are useful for: <br>
- Breaking up a complex expression into multiple parts, each represented by a variable. 
- Defining constants outside of the query body for readability. 
- Defining a variable once and using it multiple times within a query. 

For example, you can use 2 **'let'** statements to create "LogType" and "TimeBucket" variables with the following values:
- LogType = 'Warning'
- TimeBucket = 1m

And then craft a query that performs a count of "Warning" by 1 minute Timestamp buckets (bins).
-  Remember to include a ";" at the end of your let statement.

Example:
```
let LogType='Warning';
let TimeBucket=1m;
ingestionLogs
| where Level==LogType
| summarize count() by bin(Timestamp,TimeBucket)
```

Reference:
- [let](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement#examples)
- [bin()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/binfunction)

Now, let's see how we can use _let_ as a subquery. <br>
You want to investigate if there were any exceptions thrown by the node(s) that ingested the top 3 largest files. <br>
Write a query that retrieves the exception text.

hint:
```
let NodesOfTop3Files =  
logsRaw
| where Component == "INGESTOR_EXECUTER"
| extend fileSize=tolong(Properties.size)
| top 3 by ***** 
| project *****;
logsRaw 
| where Node in (NodesOfTop3Files)
| where Message startswith "Exception"
```

---
#### Challenge 7, Task 2: Use the search operator 🎓
You received an alert early in the morning regarding multiple Timeouts in your system. You want to quickly search the traces without using specific columns or table names. Write a query to "search" for "Exception=System.Timeout" string in entire database.

Reference:
[search operator](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/searchoperator?pivots=azuredataexplorer)
 
---
#### Challenge 7, Task 3: Parse Key-Value pairs strings into separate columns 🎓
As part of an incident investigation, you need to look at the INGESTOR_GATEWAY records (Component == 'INGESTOR_GATEWAY'). <br>
You need to use the _Message_ column, which contains the message of the trace, representing the information in a key/value form.  <br>
An example of a typical message would be:
```
$$IngestionCommand table=scaleEvents format=json
```
You want to analyze all the message strings, by extracting the _Message_ text into 2 calculated separate columns: table and format. 
Let's extract that to discover the number of records per format (_summarize count() by format_). <br> 
Hint: Don't think about writing a REGEX. ADX can parse key-value pairs, JSON, XML, URLs, and IPv4s. 

Reference:
[parse-kv](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/parse-kv-operator)
 
---
#### Timeseries Analytics and Machine Learning with Azure Data Explorer

Many interesting use cases use machine learning algorithms and derive interesting insights from telemetry data. Often, these algorithms require a strictly structured dataset as their input. The raw log data usually doesn't match the required structure and size. We will see how we can use the make-series operator to create well curated data (time series).

Then, we can use built in functions like [series_decompose_anomalies](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction). Anomalies/ outliers will be detected by the Kusto service and highlighted as red dots on the time series chart.

**Time series - What is it?**

A time series is a collection of observations of well-defined data items obtained through repeated measurements over time and listed in time order. Most commonly, the data points are consistently measured at equally spaced intervals. For example, measuring the temperature of the room each minute of the day would comprise a time series. Data collected irregularly is not a time series.

**What is time series analysis?**

Time series analysis comprises methods for analyzing time series data in order to extract meaningful statistics and other characteristics of the data. Time series forecasting, for example, is the use of a model to predict future values based on previously observed values. 

**What is time series decomposition?**

Time series decomposition involves thinking of a series as a combination of 4 components: 
- trends (increasing or decreasing value in the series)
- seasonality (repeating short-term cycle in the series)
- baseline (the predicted value of the series, which is the sum of seasonal and trend components) 
- noise (the residual random variation in the series). 
We can use built in functions, that uses time series decomposition to forecast future metric values and/or detect anomalous values.

**Why should you use series instead of the summarize operator?**
The summarize operator does not add "null bins" — rows for time bin values for which there's no corresponding row in the table. It's a good idea to "pad" the table with those bins. Advanced built in ML capabilities like anomaly detection need the data points to be **consistently measured at equally spaced intervals**. <br>
The _make-series_ operator can create such a “complete” series.

---

#### Challenge 7, Task 4: Nulls are important in timeseries analysis (Compare summarize and make-series)

In this task, you are going to compare the output of 2 different operators (summarize & make-series) within KQL for the same query. This task is divided into 2 parts. You will compare the outputs of these 2 parts.

Part 1: Calculate the average size of data ingested per 30 min by the node 'Engine000000000378'. Use Component as 'INGESTOR_EXECUTER'. File size is available in the 'Properties' column. Render the output as a timechart.

Hint 1: Use 'extend' operator to create a calculated/derived column <br>
Hint 2: Think about the datatype of the calculated column. Use 'tolong()'.<br>
Hint 3: Use bin() function to create time buckets of the specified duration<br>
Hint 4: Use 'summarize' operator to do the calculation

Example Output:
![Screen capture 1](/assets/images/Challenge7-Task4-Part1-Pic1.png)

Reference:
- [summarize operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/summarizeoperator)
- [extend operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/extendoperator)
- [bin()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/binfunction)
- [tolong()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/tolongfunction)

Part 2: Calculate the average size of data ingested per 30 min by the node 'Engine000000000378'. Use Component as 'INGESTOR_EXECUTER'. File size is available in the 'Properties' column. Render it as a timechart.

Hint 1: Use 'extend' operator to create a calculated/derived column<br>
Hint 2: Think about the datatype of the calculated column. Use 'tolong()'.<br>
Hint 3: Use 'make-series' operator <br>

Example Output:
![Screen capture 1](/assets/images/Challenge7-Task4-Part2-Pic1.png)


**Why should you use make-series instead of the summarize operator?**

The summarize operator does not add "null bins" — rows for time bin values for which there's no corresponding row in the table. It's a good idea to "pad" the table with those bins. Advanced built in ML capabilities like anomaly detection need the data points to be consistently measured at equally spaced intervals. The **make-series** can create such a “complete” series.

Reference:
- [extend operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/extendoperator)
- [tolong()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/tolongfunction)
- [make-series](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/make-seriesoperator)
- [Time series analysis](https://learn.microsoft.com/en-us/azure/data-explorer/time-series-analysis)

---

#### Challenge 7, Task 5: Anomaly detection 🎓
Anomaly detection lets you find outliers/anomalies in the data. <br>
Let's find out any file size anomalies by summarizing the sum of file sizes in 5-minute intervals, using the 'logsRaw' table, filtered by INGESTOR_EXECUTER _Component_ only <br>
Can you spot red dots indicating outliers/anomalies i.e.,spikes in file size on the chart?

Hint: Use series_decompose_anomalies to render anomaly chart<br>
Hint:
```
logsRaw 
| where Component == "INGESTOR_EXECUTER"
| extend fileSize=tolong(Properties.size)
| make-series records_series=sum(fileSize) on Timestamp step 5min 
| extend ActualUsage = ..............
| render anomalychart with(anomalycolumns =ActualUsage, title="file size anomalies")
```

Example result:<br>
<img src="/assets/images/anomalies.png" width="1100">

Reference:
[ADX Anomaly Detection](https://docs.microsoft.com/en-us/azure/data-explorer/anomaly-detection#time-series-anomaly-detection)<br>
[Series Decompose Anomalies](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction)


**How to display the anomalies in a tabular format?**<br>
The _series_decompose_anomalies_ function returns the following respective series:

- ad_flag: A three-part series containing the values (+1, -1, 0) marking up/down/no anomaly respectively
- ad_score: Anomaly score (using Tukey's fence test. Anomaly scores above 1.5 or below -1.5 indicate a mild anomaly rise or decline respectively. Anomaly scores above 3.0 or below -3.0 indicate a strong anomaly)
- baseline: The predicted value of the series, according to the decomposition

To get a tabular format of the detected anomalies, you can use the _mv-expand_ operator to expand the multi-value dynamic array of the anomaly detection component (AnomalyFlags, AnomalyScore, PredictedUsage) into multiple match records, and then filter by positive and negative deviations from expected usage (where AnomalyFlags != 0). <br>
 Example:
```
logsRaw 
| where Component == "INGESTOR_EXECUTER"
| extend fileSize=tolong(Properties.size)
| make-series ActualSize=sum(fileSize) on Timestamp step 5min // Creates the time series, listed by data type
| extend(AnomalyFlags, AnomalyScore, PredictedSize) = series_decompose_anomalies(ActualSize, -1) // Scores and extracts anomalies based on the output of make-series 
| mv-expand ActualSize to typeof(double), Timestamp to typeof(datetime), AnomalyFlags to typeof(double),AnomalyScore to typeof(double), PredictedSize to typeof(long) // Expands the array created by series_decompose_anomalies()
| where AnomalyFlags != 0  // Returns all positive and negative deviations from expected usage
| project Timestamp,ActualSize = format_bytes(ActualSize, 2),PredictedSize = format_bytes(PredictedSize, 2), AnomalyScore, AnomalyFlags // Defines which columns to return 
| sort by abs(AnomalyScore) desc // Sorts results by anomaly score in descending ordering
```
<img src="/assets/images/anomalies_table.png" width="800">

Looking at the query results, you can see that the query: <br>
- Calculates an expected sum (of the file size) for each bucket.
- Compares actual size to expected size.
- Assigns an anomaly score to each data point, indicating the extent of the deviation of actual size from expected size.
- Identifies positive (1) and negative (-1) anomalies.

---
### Challenge 8: Visualization

Using the Dashboard feature of Azure Data Explorer, build a dashboard using outputs of below 3 queries (on ingestionLogs table).

When creating a dashboard in ADX, use the cluster URI of your free cluster as the data source.
<img src="(https://github.com/Azure/ADX-in-a-Day-Lab1/raw/main/assets/images/free_cluster_create_db.png)" width="800">

Try this! Create a Timechart using following query. Observe that we used _startTime and _endTime. These 2 are parameters from TimeRange filter in ADX Dashboard with which we can filter the minimum and maximum time of our data.
```
ingestionLogs
| where Timestamp between (todatetime(_startTime) .. todatetime(_endTime))
| summarize count() by bin(Timestamp, 10m), Component
```
- Use the above example query as reference to add Timestamp filter with _startTime and _endTime filter to queries in task 1 and task 2.

- The following 2 tasks use the timefilter between 2014-03-08T07:00:00 and 2014-03-08T12:00:00

---
#### Challenge 8, Task 1 : Find the anomaly value 🎓
Parameterize (add Timefilter) and render an Anomaly chart using the following Anomaly detection query. The chart should show values between 2014-03-08T07:00:00 and 2014-03-08T12:00:00.
**Question**: What is the anomaly value(y axis) at exactly 10:49 on x axis.

```
let TimeBuckets = 1m;
ingestionLogs 
| where Level == "Information"
| make-series MySeries=count() on Timestamp step TimeBuckets by Level
| extend anomaly = series_decompose_anomalies(MySeries)
```
---
#### Challenge 8, Task 2 : Find the warning percentage 🎓
Parameterized (add Timefilter) and render a Piechart using the following query. The chart should show values between 2014-03-08T07:00:00 and 2014-03-08T12:00:00.

**Question**: What is the warning % on the piechart?

```
ingestionLogs
| summarize count() by Level
```

You can directly add a query from query window to an existing dashboard.
Hint 1: In the query window, explore the “Share” menu.
  
  ![Screen capture 1](/assets/images/Challenge8-Task1-Pic1.png)

Reference:
- [Visualize data with the Azure Data Explorer dashboard](https://docs.microsoft.com/en-us/azure/data-explorer/azure-data-explorer-dashboards)
- [Parameters in Azure Data Explorer dashboards](https://docs.microsoft.com/en-us/azure/data-explorer/dashboard-parameters)

Note: Below is just an example dashboard

<img src="/assets/images/Challenge8-Task1-dashboard-v2.png" width="800">

  
--- 
### Up for more challenges?
#### Challenge 9: Prepare management dashboard with PowerBI
Visualize the outputs of any 2 queries in PowerBI using the DirectQuery mode. 

There are multiple ways to connect ADX and PowerBI depending on the use case. 

Reference:
- [Visualize data using the Azure Data Explorer connector for Power BI](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-connector)
- [Visualize data using a query imported into Power BI](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-imported-query)

---
### Continue your learning journey!
#### - Learn and get hands on experience with a full blown ADX cluster and advanced ADX topics like Materialized Views, scaling, security, geo mapping and more. 
[Azure Data Explorer Microhack](https://github.com/Azure/azure-kusto-microhack)

<img src="/assets/images/microhack_badge.png" width="200">

#### - Become a detective and solve some puzzles using Kusto Query Langugage! You can reuse the same free cluster that you have used to complete ADX-in-a-Day challenges.
[Kusto Detective Agency](https://detective.kusto.io)

<img src="/assets/images/kda_badge.png" width="200">

---

🎉 Congrats! You've completed the ADX in a Day Lab 2 challenges! <br>
To earn the digital badge, submit the KQL queries/commands of the challenges marked with 🎓: [Answer sheet - ADX in a Day Lab 2](https://forms.office.com/r/J38XTAx09Q)

<details>
<summary><b>Contributing</b></summary><br>

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
</details>
  
<details>
<summary><b>Trademarks</b></summary><br>

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
</details>
