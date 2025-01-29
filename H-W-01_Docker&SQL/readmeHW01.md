# Homework 01: Docker & SQL

Environment on which these commands were executed:
+ OS: Windows 11
+ WSL2 version: `PS C:\Users\Adrian> wsl --version`
```
PS C:\Users\Adrian> wsl --version
Versión de WSL: 2.3.26.0
Versión de kernel: 5.15.167.4-1
Versión de WSLg: 1.0.65
Versión de MSRDC: 1.2.5620
Versión de Direct3D: 1.611.1-81528511
Versión DXCore: 10.0.26100.1-240331-1435.ge-release
Versión de Windows: 10.0.26100.2894
```
+ Anaconda version: `$ conda -V`
```
conda 24.9.2
```
From this, all was executed in a conda environment with:
+ Python version: `$ python -V`
```
Python 3.12.7
```
+ Docker version: `$ docker -v`
```
Docker version 26.1.3, build 26.1.3-0ubuntu1~24.04.1
```

## Question N°1

> Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash. What's the version of pip in the image?

To do this step, you must to run the command: `docker run --entrypoint=bash -it python:3.12.8`

After docker run the image and show you the bash of the container, execute: `pip -V`
```
root@4cbc8a01ac2f:/# pip -V
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```

## Question N°2
> Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?

Is `db:5432` because the elements inside the docker compose "environment" can see each other and the ports mapped from "inside", so `pgadmin` can see the internal port of the `db` that is `5432` defined in the line `- '5433:5432'`. The port `5433` it is available outside the docker compose container.  

## Question N°3
> During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:
> + Up to 1 mile
> + In between 1 (exclusive) and 3 miles (inclusive),
> + In between 3 (exclusive) and 7 miles (inclusive),
> + In between 7 (exclusive) and 10 miles (inclusive),
> + Over 10 miles

To do this, first I ingest data related with the green taxi trips in NY. More info [HERE](https://github.com/adrianvaszul/DataEngineZoomcamp/blob/main/H-W-01_Docker%26SQL/ingest.md).

Then I executed this SQL query:

```sql
SELECT COUNT(*) AS "Up to 1 mile"
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_dropoff_datetime < '2019-11-01'
AND trip_distance <= 1;

SELECT COUNT(*) AS "1 to 3 miles"
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_dropoff_datetime < '2019-11-01'
AND trip_distance > 1 AND trip_distance <= 3;

SELECT COUNT(*) AS "3 to 7 miles"
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_dropoff_datetime < '2019-11-01'
AND trip_distance > 3 AND trip_distance <= 7;

SELECT COUNT(*) AS "7 to 10 miles"
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_dropoff_datetime < '2019-11-01'
AND trip_distance > 7 AND trip_distance <= 10;

SELECT COUNT(*) AS "Over 10 miles"
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_dropoff_datetime < '2019-11-01'
AND trip_distance > 10;
```
And the output was:

```
+--------------+
| Up to 1 mile |
|--------------|
| 104802       |
+--------------+
SELECT 1
+--------------+
| 1 to 3 miles |
|--------------|
| 198924       |
+--------------+
SELECT 1
+--------------+
| 3 to 7 miles |
|--------------|
| 109603       |
+--------------+
SELECT 1
+---------------+
| 7 to 10 miles |
|---------------|
| 27678         |
+---------------+
SELECT 1
+---------------+
| Over 10 miles |
|---------------|
| 35189         |
+---------------+
```

## Question N°4

> Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

The SQL query was:
```sql
SELECT lpep_pickup_datetime, trip_distance
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-11'  AND lpep_pickup_datetime < '2019-10-12'
ORDER BY trip_distance DESC
LIMIT 1;

SELECT lpep_pickup_datetime, trip_distance
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-24'  AND lpep_pickup_datetime < '2019-10-25'
ORDER BY trip_distance DESC
LIMIT 1;

SELECT lpep_pickup_datetime, trip_distance
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-26'  AND lpep_pickup_datetime < '2019-10-27'
ORDER BY trip_distance DESC
LIMIT 1;

SELECT lpep_pickup_datetime, trip_distance
FROM green_trip_2019
WHERE lpep_pickup_datetime >= '2019-10-31'  AND lpep_pickup_datetime < '2019-11-01'
ORDER BY trip_distance DESC
LIMIT 1;
```
And the output:
```
+----------------------+---------------+
| lpep_pickup_datetime | trip_distance |
|----------------------+---------------|
| 2019-10-11 20:34:21  | 95.78         |
+----------------------+---------------+
SELECT 1
+----------------------+---------------+
| lpep_pickup_datetime | trip_distance |
|----------------------+---------------|
| 2019-10-24 10:59:58  | 90.75         |
+----------------------+---------------+
SELECT 1
+----------------------+---------------+
| lpep_pickup_datetime | trip_distance |
|----------------------+---------------|
| 2019-10-26 03:02:39  | 91.56         |
+----------------------+---------------+
SELECT 1
+----------------------+---------------+
| lpep_pickup_datetime | trip_distance |
|----------------------+---------------|
| 2019-10-31 23:23:41  | 515.89        |
+----------------------+---------------+
```
Here we can see clearly that the day `2019-10-31` was the longest trip with `515.89` miles.

## Question N°5
> Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18? \
> Consider only lpep_pickup_datetime when filtering by date.

The SQL query was:
```sql
SELECT green_trip_2019."PULocationID", taxi_zone."Zone", SUM(total_amount) AS "Total_Paid"
FROM green_trip_2019
JOIN taxi_zone ON green_trip_2019."PULocationID" = taxi_zone."LocationID"
WHERE lpep_pickup_datetime >= '2019-10-18'  AND lpep_pickup_datetime < '2019-10-19'
GROUP BY "PULocationID", taxi_zone."Zone"
HAVING SUM(total_amount) >= 13000
ORDER BY "Total_Paid" DESC
LIMIT 5;
```
And the output:
```
+--------------+---------------------+--------------------+
| PULocationID | Zone                | Total_Paid         |
|--------------+---------------------+--------------------|
| 74           | East Harlem North   | 18686.68000000006  |
| 75           | East Harlem South   | 16797.260000000075 |
| 166          | Morningside Heights | 13029.790000000034 |
+--------------+---------------------+--------------------+
```
This is enough to answered the question.

## Question N°6
> For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip?

The SQL query was:
```sql
SELECT green_trip_2019."PULocationID", zoneini."Zone" AS "CityPU",
green_trip_2019."DOLocationID", zoneend."Zone" AS "CityDO",
tip_amount
FROM green_trip_2019
JOIN taxi_zone zoneini ON green_trip_2019."PULocationID" = zoneini."LocationID"
JOIN taxi_zone zoneend ON green_trip_2019."DOLocationID" = zoneend."LocationID"
WHERE lpep_pickup_datetime >= '2019-10-01'  AND lpep_pickup_datetime < '2019-11-01'
AND zoneini."Zone" = 'East Harlem North'
ORDER BY "tip_amount" DESC
LIMIT 5;
```
And the output:
```
+--------------+-------------------+--------------+-------------------+------------+
| PULocationID | CityPU            | DOLocationID | CityDO            | tip_amount |
|--------------+-------------------+--------------+-------------------+------------|
| 74           | East Harlem North | 132          | JFK Airport       | 87.3       |
| 74           | East Harlem North | 263          | Yorkville West    | 80.88      |
| 74           | East Harlem North | 74           | East Harlem North | 40.0       |
| 74           | East Harlem North | 74           | East Harlem North | 35.0       |
| 74           | East Harlem North | 1            | Newark Airport    | 26.45      |
+--------------+-------------------+--------------+-------------------+------------+
```
We can see that `JFK Airport` have the largest tip with the corresponding conditions (picked up in October 2019 and started on East Harlem North)
