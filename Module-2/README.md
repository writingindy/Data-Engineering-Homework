## Module 2 Homework: Workflow Orchestration

### Question 1
I set `disabled` to `true` in Kestra under `purge_files` and then executed it with yellow, year 2020, and month 12. Then I looked at the outputs and got the file size for the .csv file. 


```
- id: purge_files
    type: io.kestra.plugin.core.storage.PurgeCurrentExecutionFiles
    description: If you'd like to explore Kestra outputs, disable it.
    disabled: true
```

The answer is 128.3 MB.

### Question 2
To get the rendered value, I just substitute what the inputs are into the variable `file`.

The answer is green_tripdata_2020-04.csv.

### Question 3
Using `06_gcp_taxi_scheduled` flow and scheduling a backfill in Kestra, I ran:

```
SELECT * FROM `kestra-sandbox-450021.zoomcamp.yellow_tripdata` WHERE tpep_pickup_datetime >= '2020-01-01' AND tpep_pickup_datetime < '2021-01-01' ORDER BY tpep_pickup_datetime;
```
on BigQuery in GCP.

The answer is 24,648,499.

### Question 4
Using `06_gcp_taxi_scheduled` flow and scheduling a backfill in Kestra, I ran:

```
SELECT * FROM `kestra-sandbox-450021.zoomcamp.green_tripdata` WHERE lpep_pickup_datetime >= '2020-01-01' AND lpep_pickup_datetime < '2021-01-01' ORDER BY lpep_pickup_datetime;
```
on BigQuery in GCP.

The answer is 1,734,051.

## Question 5
Using `06_gcp_taxi` flow in Kestra to execute for the Yellow taxi data and March 2021, I ran:
```
SELECT * FROM `kestra-sandbox-450021.zoomcamp.yellow_tripdata` WHERE tpep_pickup_datetime >= '2021-03-01' AND tpep_pickup_datetime < '2021-04-01' ORDER BY tpep_pickup_datetime;
```
on BigQuery in GCP.

The answer is 1,925,152.

## Question 6
I google `timezone Schedule trigger Kestra` and found this page: https://kestra.io/plugins/core/triggers/io.kestra.plugin.core.trigger.schedule

The answer is to add a `timezone` property set to `America/New_York` in the `Schedule` trigger config.


