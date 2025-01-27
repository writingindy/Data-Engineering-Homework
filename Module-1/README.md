## Module 1 Homework: Docker & SQL

### Question 1
To run docker with the `python:3.12.8` image in interactive mode, using the `bash` entrypoint:
```
(base) indy@indy-ThinkPad-P14s-Gen-4:~/Desktop/Data-Engineering-zoomcamp/Module1$ sudo docker run -it --entrypoint=bash python:3.12.8
Unable to find image 'python:3.12.8' locally
3.12.8: Pulling from library/python
fd0410a2d1ae: Pull complete 
bf571be90f05: Pull complete 
684a51896c82: Pull complete 
fbf93b646d6b: Pull complete 
12f3828c4288: Pull complete 
4d8be491b866: Pull complete 
ec162e081748: Pull complete 
Digest: sha256:2e726959b8df5cd9fd95a4cbd6dcd23d8a89e750e9c2c5dc077ba56365c6a925
Status: Downloaded newer image for python:3.12.8
root@3ffada85ebda:/# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```

### Question 2
The `hostname` and `port` pgadmin should use to connect to the postgres database is:
- hostname: db
- port: 5432

### Prepare Postgres
We first run the commands:
```
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```
and extract the `green_tripdata_2019-10.csv.gz`.

Then we run:
```
sudo docker run -it -e POSTGRES_USER="root" -e POSTGRES_PASSWORD="root" -e POSTGRES_DB="ny_taxi" -v /home/indy/Desktop/Data-Engineering-zoomcamp/Data-Engineering-Homework/Module-1/ny_taxi_postgres_data:/var/lib/postgresql/data -p 5432:5432 --network=pg-network --name pg-database postgres:13
```
then run code in `upload-data.ipynb` and `upload-zone-data.ipynb` to put the data into postgres.

Then we start a pgadmin instance to be able to run SQL commands on the datasetL
```
sudo docker run -it \
-e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
-e PGADMIN_DEFAULT_PASSWORD="root" \
-p 8080:80 \
--network=pg-network \
--name pg-admin \
dpage/pgadmin4
```


### Question 3
Part 1
```
SELECT COUNT(trip_distance)
FROM green_taxi_data AS g
WHERE 
g.lpep_pickup_datetime >= '2019-10-01'
AND
g.lpep_dropoff_datetime < '2019-11-01'
AND
g.trip_distance <= 1;
```

Part 2
```
SELECT COUNT(trip_distance)
FROM green_taxi_data AS g
WHERE 
g.lpep_pickup_datetime >= '2019-10-01'
AND
g.lpep_dropoff_datetime < '2019-11-01'
AND
g.trip_distance > 1
AND
g.trip_distance <= 3;
```

Part 3
```
SELECT COUNT(trip_distance)
FROM green_taxi_data AS g
WHERE 
g.lpep_pickup_datetime >= '2019-10-01'
AND
g.lpep_dropoff_datetime < '2019-11-01'
AND
g.trip_distance > 3
AND
g.trip_distance <= 7;
```

Part 4
```
SELECT COUNT(trip_distance)
FROM green_taxi_data AS g
WHERE 
g.lpep_pickup_datetime >= '2019-10-01'
AND
g.lpep_dropoff_datetime < '2019-11-01'
AND
g.trip_distance > 7
AND
g.trip_distance <= 10;
```

Part 5
```
SELECT COUNT(trip_distance)
FROM green_taxi_data AS g
WHERE 
g.lpep_pickup_datetime >= '2019-10-01'
AND
g.lpep_dropoff_datetime < '2019-11-01'
AND
g.trip_distance > 10;
```

### Question 4
First we run:
```
SELECT MAX(trip_distance)
FROM green_taxi_data;
```
then
```
SELECT lpep_pickup_datetime
FROM green_taxi_data
WHERE
trip_distance = 515.89;
```

### Question 5
```
SELECT "Zone", SUM(total_amount) AS sum_money
FROM green_taxi_data g LEFT JOIN green_taxi_lookup l
ON g."PULocationID" = l."LocationID"
WHERE
lpep_pickup_datetime >= '2019-10-18'
AND
lpep_pickup_datetime < '2019-10-19'
GROUP BY "Zone"
ORDER BY sum_money DESC;
```

There was a NULL at the top, so I double checked by looking at:
```
SELECT "PULocationID", SUM(total_amount) AS sum_money
FROM green_taxi_data g LEFT JOIN green_taxi_lookup l
ON g."PULocationID" = l."LocationID"
WHERE
lpep_pickup_datetime >= '2019-10-18'
AND
lpep_pickup_datetime < '2019-10-19'
GROUP BY "PULocationID"
ORDER BY sum_money DESC;
```
and manually checking the PULocationID in `taxi_zone_lookup.csv`.

### Question 6
```
SELECT MAX(tip_amount) as max_tip, "DOLocationID", l2."Zone"
FROM green_taxi_data g LEFT JOIN green_taxi_lookup l
ON g."PULocationID" = l."LocationID" 
LEFT JOIN green_taxi_lookup AS l2 ON g."DOLocationID" = l2."LocationID"
WHERE
lpep_pickup_datetime >= '2019-10-01'
AND
lpep_pickup_datetime < '2019-11-01'
AND
l."Zone" = 'East Harlem North'
GROUP BY
"DOLocationID", l2."Zone"
ORDER BY max_tip DESC
;
```

Another NULL for the answer, so we consult the `taxi_zone_lookup.csv` again.

