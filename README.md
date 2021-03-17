## TensorBase Frontier Edition

New Bigdata Warehousing for SMEs (Small and Medium Enterprises)

Read more about TensorBase Frontier Edition from the [website](https://tensorbase.io/).

The alpha plan of TensorBase FE is in progress. If you are interested to preview, go to the [website](https://tensorbase.io/) for free application.

An open source TensorBase Base Edition is also in progress. If you are a developer and interesting, go to the [TensorBase BE](https://github.com/tensorbase/tensorbase) project.

More infos about TensorBase FE will be provided gradually.

## Roadmap
- [x] Alpha
    + [x] single-table SQL operations with ClickHouse compatibility
- [] Beta
    + [] BaseStorage with advanced functionalities
    + [] Join (all primary SQL operations done)
    + [] High availability and reliability on multi-devices in a single node (dedicate introduction when its ready)

## User Guide

## Benchmarks

#### Setup
----------

* Hardware: 1 Socket, Intel Xeon Platinum 8260, 24 cores / 48 hyperthreads, 6-channel DDR4-2400 ECC REG DRAMs
* TensorBase Frontier Edition (2021.3.0.a) v.s. ClickHouse server v21.2.5.5-stable
* Measurement rules: repeat 5 times for every query and pick up the shortest end-to-end query time(shown in the clickhouse-client command)
* Measurements in the table: end-to-end time(in seconds)/end-to-end bandwidth(in GB/s) for big dataset, end-to-end time(in seconds) for others
* The parallelism of ClickHouse query is set to <b>48</b>. Note: it is observed that the higher threads setting (but <= max hardware threads) may degrade the performance of ClickHouse in some extent. However, we still use 48-threads setting, because: 
1. hardware under-utilization (if have) is a flaw for performance oriented systems
2. performance degradation in higher threads (if have) is a flaw in actual concurrent production environment
 
<br/>

### Performance #0 - system.numbers
------------------------------------

* Dataset: 1000,000,000,000 (1 Trillion or 1000 Billion)

|Query |ClickHouse (v21.2.5.5)      | TensorBase FE (2021.3.0.a)  | Speedup Ratio  of TB FE  |
|------|--------------------------------| ----------------------- | -------------------------- |
|SELECT sum(number) FROM system.numbers | 28.6 sec / 279.4 GB/s   |  0.027 sec / ~ | 1059x |
|SELECT max(number) FROM system.numbers | 51.9 sec / 154.0 GB/s   |  0.027 sec / ~ |  1922x |
|SELECT max(123*number+456*number+789*number) FROM system.numbers | 303.363 sec / 26.37 GB/s |  0.028 sec / ~ | 10833x |

Note:
* Use table system.numbers_mt in ClickHouse, system.numbers in TensorBase FE
* In ClickHouse, system.numbers is a kind of virtual table to represent the natural number dataset. The measurements for system.numbers/numbers_mt makes no senses for the real world, but still uncovers the unique of TensorBase. And you may accidentally meet some other compares this kind non-senses, here you seen, TensorBase also has and run in a tiny constant time.  

<br/>

### Performance #1 - medium dataset
------------------------------------

ClickHouse create table script:
```sql
CREATE TABLE default.trips_lite_n10m
(
    trip_id UInt32,
    pickup_datetime DateTime
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(pickup_datetime)
ORDER BY trip_id
```
TensorBase FE  create table script:

```sql
CREATE TABLE default.trips_lite_n10m
(
    trip_id UInt32,
    pickup_datetime DateTime
)
ENGINE = BaseStorage
PARTITION BY toYYYYMM(pickup_datetime)
```

* NYC Taxi lite (Stripped) Dataset[1]: 10 million taxi trips data

|Query |ClickHouse (v21.2.5.5)      | TensorBase FE (2021.3.0.a)  | Speedup Ratio  of TB FE  |
|------|--------------------------------| ----------------------- | -------------------------- |
|select count(trip_id) from trips_lite_n10m | 0.021 sec |  0.003 sec | 7x |
|select sum(123*trip_id+456) from trips_lite_n10m where trip_id>666 | 0.033 sec |  0.027 sec |  1.2x |
|select toYear(pickup_datetime), sum(trip_id) from trips_lite_n10m group by toYear(pickup_datetime) order by toYear(pickup_datetime) | 0.041 sec |  0.031 sec | 1.3x |


<br/>

### Performance #2 - big dataset
---------------------------------

ClickHouse create table script:
```sql
CREATE TABLE default.trips_lite
(
    trip_id UInt32,
    pickup_datetime DateTime
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(pickup_datetime)
ORDER BY trip_id
```
TensorBase FE  create table script:

```sql
CREATE TABLE default.trips_lite
(
    trip_id UInt32,
    pickup_datetime DateTime
)
ENGINE = BaseStorage
PARTITION BY toYYYYMM(pickup_datetime)
```

* NYC Taxi Dataset[2]: 1.47 billion taxi trips data

|Label|Query |ClickHouse (v21.2.5.5)      | TensorBase FE (2021.3.0.a)  | Speedup Ratio  of TB FE  |
|------|------|--------------------------------| ----------------------- | -------------------------- |
|Q#1|select count(trip_id) from trips_lite | 0.251 sec / 23.35 GB/s   | 0.008 sec / ~ | 31.4x |
|Q#2|select sum(123*trip_id+456) from trips_lite where trip_id>666 | 0.898 sec / 6.52 GB/s | 0.072 sec / 81.4 GB/s  | 12.5x |
|Q#3|select count(pickup_datetime), sum(trip_id+1000) from trips_lite | 0.849 sec / 13.81 GB/s | 0.072 sec / ~ | 11.8x |
|Q#4|select toYear(pickup_datetime), sum(trip_id) from trips_lite group by toYear(pickup_datetime) order by toYear(pickup_datetime) | 0.933 sec / 12.56 GB/s |  0.176 sec / 66.6 GB/s | 5.3x  |
|Q#5|select toYYYYMM(pickup_datetime), sum(trip_id) from trips_lite WHERE trip_id > 100000 group by toYYYYMM(pickup_datetime) order by toYYYYMM(pickup_datetime) |  1.186 sec /  9.88 GB/s |  0.261 sec / 44.9 GB/s | 4.5x |


<br/>

### Performance #3 - big dataset
----------------------------------

* TPC-H Dataset 

(Coming soon...)

<br/>

## References
-------------

1. [Stripped NYC Taxi lite Dataset in CSV](/trips_lite_n10m.tar.xz)
2. [Googling NYC Taxi Dataset](https://www.google.com/search?q=NYC+TAXI+Dataset)