# Ingest Data

Some additional information: https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf

To ingest data to the postgres, I downloaded two files with these commands:

`wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz`

`wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv`

Then I executed the next python script called `Step_toSQL.py` with the followed command and arguments to ingest the file `green_tripdata_2019-10.csv`:
```
$ python Step_toSQL.py --change green_tripdata_2019-10.csv green_trip_2019
```

Lastly I executed the same python script with other arguments to ingest the file `taxi_zone_lookup.csv`:
```
$ python Step_toSQL.py taxi_zone_lookup.csv taxi_zone
```
With this I successfully loaded two tables to postgres:
```
root@localhost:ny_taxi> \d
+--------+-----------------+-------+-------+
| Schema | Name            | Type  | Owner |
|--------+-----------------+-------+-------|
| public | green_trip_2019 | table | root  |
| public | taxi_zone       | table | root  |
+--------+-----------------+-------+-------+
```
with the followed size:
```
root@localhost:ny_taxi> SELECT COUNT(*) AS nrows_zone FROM taxi_zone; SELECT COUNT(*) AS nrows_taxi FROM green_trip_
 2019
+------------+
| nrows_zone |
|------------|
| 265        |
+------------+
SELECT 1
+------------+
| nrows_taxi |
|------------|
| 476386     |
+------------+
```

### Step_toSQL.py
```python
import pandas as pd
from sqlalchemy import create_engine
from time import time
import argparse

CHUNK=100000


def ingest(params): 
    path=params.path
    table=params.table
    ChangeTime=params.change
    
    print(f"Starting the script...")
    engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
    df=pd.read_csv(path, nrows=10)
    if ChangeTime == True:
        df.lpep_pickup_datetime = pd.to_datetime(df.lpep_pickup_datetime)
        df.lpep_dropoff_datetime = pd.to_datetime(df.lpep_dropoff_datetime)
    pd.io.sql.get_schema(df, name=table, con=engine)
    df.head(n=0).to_sql(name=table, con=engine , if_exists='replace')
    print(f"Schema loaded")

    total = 0
    print(f"Start inserting chucks with {CHUNK} size.")
    for i, lote in enumerate(pd.read_csv(path, iterator=True, low_memory=False, chunksize=CHUNK),1):
        try:
            t_start = time()
            if ChangeTime == True:
                lote.lpep_pickup_datetime = pd.to_datetime(lote.lpep_pickup_datetime)
                lote.lpep_dropoff_datetime = pd.to_datetime(lote.lpep_dropoff_datetime)
            lote.to_sql(name=table, con=engine , if_exists='append')
            t_end = time()
            total += len(lote)
            print(f"insterted chunck {i}. Total: {total}. took %.3f seconds" % (t_end-t_start))
            
        except Exception as e:
            print(f"Error en lote {i}: {e}")    


if __name__ == '__main__':

    parser = argparse.ArgumentParser(description='Ingest data')
    parser.add_argument('path', help='path of the location of csv') 
    parser.add_argument('--change', help='if you decide change the datetime', action="store_true") 
    parser.add_argument('table', help='Table name where to store') 
    
    args = parser.parse_args()

    ingest(args)

```
