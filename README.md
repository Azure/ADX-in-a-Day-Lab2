# Lab 2: Advanced KQL, Policies, and Visualization

This Lab is organised into the following 4 challenges:
|Challenge | Description | Est. Time|
|--|--|--|
| [Challenge 5](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-5-caching-and-retention-policies)| Caching and retention policies| 30 Min| 
| [Challenge 6](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-6-control-commands)| Control commands| 30 Min|
| [Challenge 7](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-7-going-more-advanced-with-kql)| Advanced KQL operators| 1 Hour|
| [Challenge 8](https://github.com/Azure/ADX-in-a-Day-Lab2#challenge-8-visualization)| Visualization| 1 Hour |


---
In order to receive the "ADX In a Day" digital badge, you will need to complete the challenges marked with 🎓. Please submit the KQL queries/commands of these challenges in the following link: [Answer sheet - ADX in a Day Lab 2](https://forms.office.com/r/Pu1AdWBDwG)
---
---
# [Go to ADX-In-A-Day Homepage](https://github.com/Azure/ADX-in-a-Day)

---
### Challenge 5: Caching and retention policies

Among the different policies you can set to the ADX cluster, two policies are of particular importance: retention policy (retention period) and cache policy (cache period).
First, a policy, is what’s  used to enforce and control the properties of the cluster (or the database/table.)
  
The **retention** policy is the time span, in days, for which it’s guaranteed that the data is kept available for querying. The time span is measured from the time that the records are ingested. When the period expires , the records  will not be available for querying any more. In other words, the retention policy defines the period in which the data available to query, measured since ingestion time. Notice that a large retention period may impact the cost. 

The **cache** policy, is the time span, in days, for which to keep recently ingested data (which is usually the frequently queried data) available in the hot cache rather than in long term storage (this is also known as cold cache. Specifically, it is Azure blob storage). Data stored in the hot cache is actually stored in local SSD or the RAM of the machine, very close to the compute nodes. Therefore, more readily available for querying. The availability of data in hot cache improves query performance but can potentially increase the cluster cost (as more data is being stored, more VMs are required to store it). In other words, the caching policy defines the period in which data is kept in the hot cache. 

All the data is always stored in the cold cache, for the duration defined in the retention policy. Any data whose age falls within the hot cache policy will also be stored in the hot cache. If you query data from cold cache, it’s recommended to target a small specific range in time (“point in time”) for queries to be efficient.

---
#### Task 1: change the retention policy via commands (data base or table level) 🎓

Database policies can be overridden per table using a KQL control command.
ADX cluster and database are Azure resources. A database is a sub-resource of the cluster, so it can be edited from the portal. Tables are not considered an Azure resource, so they cannot be managed in the portal but via a KQL command.    
You can always use KQL commands to alter the policies of the entire Cluster/Database/tables. Table level policy takes precedence over database level which takes precedence over cluster level.

Alter the retention policy of the table LogisticsTelemetryManipulated to 60 days.

[.alter table retention policy command - Azure Data Explorer | Microsoft Docs](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/alter-table-retention-policy-command)

---

### Challenge 6: Control commands

#### Task 1: .show/diagnostic logs/Insights
Control commands are requests to the service to retrieve information that is not necessarily data in the database tables, or to modify the service state, etc. In addition, they can be used to manage Azure Data Explorer.
The first character of the text of a request determines if the request is a control command or a query. Control commands must start with the dot (.) character, and no query may start by that character. <br><br>
[Management (control commands) overview](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/)

- The ‘.show queries’ command returns a list of queries that have reached a final state, and that the user invoking the command has access to see.
- The ‘.show commands' command returns a table of the admin commands that have reached a final state.  The TotalCpu columns  is the value of the total CPU clock time (User mode + Kernel mode) consumed by this command.
- The '.show journal' command returns a table that contains information about metadata operations that are done on the Azure Data Explorer database. The metadata operations can result from a control command that a user executed, or internal control commands that the system executed, such as drop extents by retention
- The '.show tables details' command returns  a set that contains the specified table or all tables in the database with a detailed summary of each table's properties.

---
#### Task 2: Use .show queries 🎓

Write a command to count the number queries that you run (use the User column), in the past 7 day.

Reference:
[.show queries](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/queries)

---
#### Task 3: Use .journal commands 🎓

Write a command to show the details on the materlized view that you created erlier. When did you create the materlized view? <br>
Hint: use the 'Event' and the 'EventTimestamp' columns.

Reference:
[.show journal](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/journal)

---
#### Task 4: Use .show commands 🎓

Write a command to count the number commands that you run (use the User column), in the past 7 day.

Reference:
[.show commands](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/commands)


---
#### Task 5: Table details and size 🎓

Write a control command to show details on all tables in the database. How many tables are in your cluster? <br>
What is the original size of the data, per table? What is the [extent](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/extents-overview) size of the data, per table <br>
Hint: | extend TotalExtentSizeInMB = format_bytes(TotalExtentSize, 0, "MB") <br>

Reference:
[.show table details](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/show-table-details-command) and [format_bytes()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/format-bytesfunction)

---

### Challenge 7: Going more advanced with KQL

#### Task 1: Declaring variables 🎓
Use a **'let'** statement to create a list of the 10 device Ids which have the highest Shock. Then, use this list in a following query to find the total average temperature of these 10 devices.

You can use the **'let'** statement to set a variable name equal to an expression or a function.
let statements are useful for:
- Breaking up a complex expression into multiple parts, each represented by a variable.
- Defining constants outside of the query body for readability.
- Defining a variable once and using it multiple times within a query.

Hint 1: [in operator - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/in-cs-operator#subquery)

Hint 2: [let - Azure Data Explorer | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement#examples)

Hint 3: Remember to include a ";" at the end of your let statement.

---
#### Task 2: Add more fields to your timechart 🎓
Write a query to show a timechart of the number of records, by TransportationMode. Use 10 minute bins.

Expected result:

 ![Screen capture 1](/assets/images/chart-4.png)


---
#### Task 3: Some geo-mapping
Write a query to show on map the locations (based on the longitude and latitude) of 10 devices with the highest temperature from the last 90 days.
<br>
Hint 1: 'top' operator </br>
Hint 2: render scatterchart with (kind = map)

Once the map is displayed, you can click on the locations. Note that in order to show more details in the balloon, you need to change the render phrase to include 'series=<TempColumn>'.

[render operator with scatter chart](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/renderoperator?pivots=azuredataexplorer)
   
  Expected result:
  
<img src="/assets/images/Challenge7-Task3-map.png" width="400">

---
#### Task 4: Range 
Range is a tabular operator: it generates a single-column table of values, whose values are start, start + step, ... up to and until stop.
Run the following query and review the results:

range MyNumbers from 1 to 8 step 2

Range also works with dates:
range LastWeek from ago(7d) to now() step 1d

We will use the range operator as part of the time series creation in the next tasks.
  
---
#### Machine learning with Kusto and time series analysis

Many interesting use cases use machine learning algorithms and derive interesting insights from telemetry data. Often, these algorithms require a strictly structured dataset as their input. The raw log data usually doesn't match the required structure and size. We will see how we can use the make-series operator to create well curated data (time series).

Then, we can use built in functions like [series_decompose_anomalies](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/series-decompose-anomaliesfunction). Anomalies/ outliers will be detected by the Kusto service and highlighted as red dots on the time series chart.

**Time series:**
What is a time series?
A time series is a collection of observations of well-defined data items obtained through repeated measurements over time and listed in time order. Most commonly, the data points are consistently measured at equally spaced intervals. For example, measuring the temperature of the room each minute of the day would comprise a time series. Data collected irregularly is not a time series.

**What is time series analysis?**

Time series analysis comprises methods for analyzing time series data in order to extract meaningful statistics and other characteristics of the data. Time series forecasting, for example, is the use of a model to predict future values based on previously observed values. 

**What is time series decomposition?**
Time series decomposition involves thinking of a series as a combination of 4 components: 
- trends (increasing or decreasing value in the series)
- seasonality (repeating short-term cycle in the series)
- baseline (the predicted value of the series, which is the sum of seasonal and trend components) 
- noise (The residual random variation in the series). 
We can use built in functions, that uses time series decomposition to forecast future metric values and/or detect anomalous values.

This is what time series looks like:

![Screen capture 1](/assets/images/Challenge7-Task4-Pic1.png)

**Why should you use series instead of the summarize operator?**

The summarize operator does not add "null bins" — rows for time bin values for which there's no corresponding row in the table. It's a good idea to "pad" the table with those bins. Advanced built in ML capabilities like anomaly detection need the data points to be consistently measured at equally spaced intervals. The **make-series** can create such a “complete” series.

---
#### Task 5: Anomaly detection 🎓
Write a query to create an anomaly chart of the average shock.

For this task, we will provide more instructions:

To generate these series, start with:
```
let min_t = (toscalar(LogisticsTelemetryHistorical | summarize min(enqueuedTime)));
let max_t = (toscalar(LogisticsTelemetryHistorical | summarize max(enqueuedTime)));
let step_interval = 10m;
LogisticsTelemetryHistorical
| make-series avg_shock_series=avg(Shock) on (enqueuedTime) from (min_t) to (max_t) step step_interval 
```
Now, we will use this avg_shock_series and run series_decompose_anomalies.
This built-in function takes an expression containing a series (dynamic numerical array) as input, and extracts anomalous points with scores.
```
| extend anomalies_flags = series_decompose_anomalies(avg_shock_series, 1) 
| render anomalychart  with(anomalycolumns=anomalies_flags, title='avg shock anomalies') 
```
The anomalies/outliers can be clearly spotted in the 'anomalies_flags' points.

[make-series](https://docs.microsoft.com/en-us/azure/data-explorer/time-series-analysis) <br>
[ADX Anomaly Detection](https://docs.microsoft.com/en-us/azure/data-explorer/anomaly-detection#time-series-anomaly-detection)
  
Expected result:
  
<img src="/assets/images/Challenge7-Task4-anomalies.png" width="650">
</br></br>

---
### Challenge 8: Visualization


#### Task 1: Prepare interactive dashboards with ADX Dashboard 🎓

Using the Dashboard feature of Azure Data Explorer, build a dashboard using outputs of any 5 queries (on LogisticsTelemetryHistorical table) that you have created in the previous challenges with the following improvements:
  - Add filter on the dashboard so that the user can choose the timespan
  - Add filter on the dashboard so that the user can choose the transportation mode

Include **filters for the dashboard** so that the queries do not need to be modified if the user wants to analyze the charts with different values of a parameter. For example, users would like to analyze the charts over the last week, the last 14 days as well as the last 1 month. Users would also like to analyze the charts by different transportation modes.

Hint 1: In the query window, explore the “Share” menu.
  
  ![Screen capture 1](/assets/images/Challenge8-Task1-Pic1.png)
 
- [Visualize data with the Azure Data Explorer dashboard | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/azure-data-explorer-dashboards)
- [Parameters in Azure Data Explorer dashboards | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/dashboard-parameters)

<img src="/assets/images/Challenge8-Task1-dashboard.png" width="500">
<img src="/assets/images/Challenge8-Task1-dashboard2.png" width="500">
  
---
#### Task 2: Prepare management dashboard with PowerBI
Visualize the outputs of any 2 queries in PowerBI using the DirectQuery mode. 

There are multiple ways to connect ADX and PowerBI depending on the use case. 

[Visualize data using the Azure Data Explorer connector for Power BI | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-connector)

[Visualize data using a query imported into Power BI | Microsoft Docs](https://docs.microsoft.com/en-us/azure/data-explorer/power-bi-imported-query)

---

🎉 Congrats! You've completed the ADX in a Day Lab 2 challenges! To earn the digital badge, submit the KQL queries/commands of the challenges marked with 🎓: [Answer sheet - ADX in a Day Lab 2](https://forms.office.com/r/Pu1AdWBDwG)

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
