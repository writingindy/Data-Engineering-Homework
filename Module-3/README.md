## Module 3 Homework: Data Warehouse

I used the provided load script to upload the dataset to a GCS Bucket. Before this, I setup a Python virtual environment and installed `google-cloud-storage`:
```
python -m venv venv
source venv/bin/activate

pip install google-cloud-storage
```
then ran the script (after filling in the relevant information) using `python load_yellow_taxi_data.py`.

I checked that the 6 files show up in the GCS bucket I created. Then I navigated to BigQuery on the Google Cloud console and created a dataset called `yellow_taxi_trip_records` in my project.

To setup an external table using the data:
```
CREATE OR REPLACE EXTERNAL TABLE `yellow_taxi_trip_records.external_yellow_tripdata`
OPTIONS (
  format= 'PARQUET',
  uris = ['gs://data-engineering-zoomcamp-hw3/yellow_tripdata_2024-*.parquet']
);
```

To create a (regular/materialized) table in BigQuery:
```
CREATE OR REPLACE TABLE `yellow_taxi_trip_records.yellow_tripdata_non_partitioned` AS
SELECT * FROM yellow_taxi_trip_records.external_yellow_tripdata;
```



### Question 1

To answer this question, I looked at the Details section of the regular table `yellow_tripdata_non_partitioned`.

The answer is 20,332,093 rows.

### Question 2

First, I turn off "Use cached results" in Query settings.

To get the estimated amount of data for a query that counts the distinct number of PULocationIDs for the **external** table, I use the following SQL query:
```
SELECT DISTINCT(PULocationID) FROM `yellow_taxi_trip_records.external_yellow_tripdata`;
```
and highlighting the query shows that this query will process 0 B when run.

For the **regular** table, I run the following SQL query:
```
SELECT DISTINCT(PULocationID) FROM `yellow_taxi_trip_records.yellow_tripdata_non_partitioned`;
```
and highlight it to see that it says the query will process 155.12 MB when run.

The answer is 0 MB for external table and 155.12 MB for materialized table.

### Question 3

The answer is because BigQuery is a columnar database and querying two columns requires reading more data than querying one column.

### Question 4

I run the following SQL query:
```
SELECT COUNT(*) FROM `yellow_taxi_trip_records.external_yellow_tripdata`
WHERE fare_amount = 0;
```
to get the answer.

The answer is 8333 records.

### Question 5

If the query will always filter based on `tpep_dropoff_datetime` and order results by `VendorID`, you should partition by `tpep_dropoff_datetime` (since BigQuery can scan partitions that match the filter and skip the remaining partitions) and cluster by `VendorID` (since this defines a column sort order using the `VendorID` column).

To create a new table with this strategy, I run:
```
CREATE OR REPLACE TABLE `yellow_taxi_trip_records.yellow_tripdata_partitioned_clustered`
PARTITION by DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `yellow_taxi_trip_records.external_yellow_tripdata`;
```

### Question 6

The query for the regular table (**without** partitioning and clustering from question 5):
```
SELECT DISTINCT(VendorID) FROM `yellow_taxi_trip_records.yellow_tripdata_non_partitioned`
WHERE tpep_dropoff_datetime > '2024-03-01' AND tpep_dropoff_datetime <= '2024-03-15';
```
and looking at the estimated bytes processed gives 310.24 MB.

For the table from question 5, running the query:
```
SELECT DISTINCT(VendorID) FROM `yellow_taxi_trip_records.yellow_tripdata_partitioned_clustered`
WHERE tpep_dropoff_datetime > '2024-03-01' AND tpep_dropoff_datetime <= '2024-03-15';
```
and looking at the estimated bytes processed gives 26.84 MB.

The answer is 310.24 MB for the regular table and 26.84 MB for the partitioned table.

### Question 7

The data in the external table would be stored in the GCP Bucket.

### Question 8

The answer for this is False - you should use clustering when it makes sense.

### BONUS QUESTION 9

The estimated number of bytes processed is 0 - this is because materialized tables pre-aggregates data (COUNT(*) is an aggregate function).