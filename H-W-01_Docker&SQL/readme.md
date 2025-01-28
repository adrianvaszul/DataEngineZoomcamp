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

## First question

> Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash. What's the version of pip in the image?

To do this step, you must to run the command: `docker run --entrypoint=bash -it python:3.12.8`

After docker run the image and show you the bash of the container, execute: `pip -V`
```
root@4cbc8a01ac2f:/# pip -V
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```

## Second question
> Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?

Is db:5433 or db:5432

## Third question
> During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:
> + Up to 1 mile
> + In between 1 (exclusive) and 3 miles (inclusive),
> + In between 3 (exclusive) and 7 miles (inclusive),
> + In between 7 (exclusive) and 10 miles (inclusive),
> + Over 10 miles

To do this, first I ingest data related with the green taxi trips in NY. More info HERE
Some additional information: https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf


```
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



