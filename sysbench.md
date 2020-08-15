# Sysbench Benchmark Testing: 

**:warning: experiment setup based on Ubuntu 16.04.**

## Prerequisites

```bash
$ sudo apt-get install automake
$ sudo apt-get install libtool
$ sudo apt-get install libmysqlclient-dev
$ sudo apt-get install libssl1.0.0 libssl-dev
```

## Installation
- Download SysBench 1.0.12 (i.e. sysbench_1.0.12.orig.tar.gz) from [Percona repositories](http://repo.percona.com/apt/pool/main/s/sysbench/).

- Untar the sysbench tar file.

```bash
$ tar -xvf sysbench_1.0.12.orig.tar.gz
```
- Go to the SysBench directory and run following commands.

```bash
$ ./autogen.sh
$ ./configure
$ make
$ make install
```

cf) If you have MySQL headers and libraries in non-standard locations, you can specify them explicitly with `--with-mysql-includes` and `--with-mysql-libs` options to `./configure` like below example.
```bash
$ ./configure --with-mysql-includes=/home/lbh/mysql-5.7.24/include --with-mysql-libs=/home/lbh/mysql-5.7.24/lib
```

Check if your mysql library is added in ```~/.bashrc```. If not, export library path. 
```bash
export LD_LIBRARY_PATH=/home/lbh/mysql-5.7.24/lib/
```
## Sysbench Usage
### Create Database
Create database for Sysbench Testing:
```bash
$ mysql -uroot

root:(none)> create database sbtest;
root:(none)> quit
```
## oltp.lua
### Sysbench Data Load

- load:
```bash
sysbench --test=/home/lbh/sysbench-1.0.12/sysbench/tests/oltp_legacy/oltp.lua --mysql-host=localhost  --mysql-db=sbtest --mysql-user=root --mysql-password=evia6587 --max-requests=0  --oltp-table-size=2000000 --max-time=600  --oltp-tables-count=200 --report-interval=10 --db-ps-mode=disable  --random-points=10 --mysql-table-engine=InnoDB --mysql-port=3306 --num-threads=128  prepare
```

### Sysbench Testing
- run:
```bash
sysbench --test=/home/lbh/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua --mysql-host=localhost  --mysql-db=sbtest --mysql-user=root --mysql-password=evia6587 --max-requests=0  --oltp-table-size=2000000 --time=259200  --oltp-tables-count=200 --report-interval=10 --db-ps-mode=disable  --random-points=10 --mysql-table-engine=InnoDB --mysql-port=3307 --mysql-socket=/tmp/mysql.sock1 --num-threads=128  run |tee /home/lbh/result/sb-ns.txt
```

## Bulk Insert 
Bulk insert does concurrent multi-low inserts, we specify how many threads we want and each one inserts into its own table. So the total number of tables is the same as the number of threads.
### InnoDB
```bash
$  sysbench bulk_insert.lua  --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=InnoDB --mysql-port=3306  prepare 
$ sysbench bulk_insert run --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=InnoDB --mysql-port=330--events=0 --time=3600
$ sysbench bulk_insert cleanup --threads=20
```
### MyRocks
```bash
$  sysbench bulk_insert.lua  --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock1 --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=rocksdb --mysql-port=3307  prepare 
$ sysbench bulk_insert run  --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock1 --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=rocksdb --mysql-port=3307 --events=0 --time=3600
$ sysbench bulk_insert cleanup --threads=20
```
## OLTP Insert
### InnoDB
```bash
$  sysbench oltp_insert.lua --threads=100 --time=3600 --table-size=5000000  --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock --mysql-user=root --mysql-password=evia6587 --mysql-port=3306  prepare 
$  sysbench oltp_insert.lua run --threads=100 --time=180 --table-size=5000000 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock --mysql-user=root --mysql-password=evia6587 --mysql-port=3306 --events=0 
$ sysbench bulk_insert cleanup --threads=100
```
### MyRocks
```bash
$  sysbench oltp_insert.lua --threads=100 --time=3600 --table-size=5000000 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock1 --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=rocksdb --mysql-port=3307  prepare 
$  sysbench oltp_insert.lua run --threads=100 --time=3600 --table-size=5000000 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock1 --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=rocksdb --mysql-port=3307 --events=0 
$ sysbench oltp_insert cleanup --threads=100
```

## OLTP Read-Only
OLTP tests try simulate transaction-oriented loads in the database, sysbench does this by running several kinds of queries inside a transaction. Below example created 10 tables, each with 10k rows, for total of 100k rows.

### Prepare 
10 tables, each with 10k rows:
```bash
$ sysbench oltp_read_only --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=InnoDB --mysql-port=3306 prepare

$ sysbench oltp_read_only --threads=20 --db-driver=mysql --mysql-db=sbtest --mysql-socket=/tmp/mysql.sock1 --mysql-user=root --mysql-password=evia6587 --mysql-table-engine=rocksdb --mysql-port=3307  prepare 
```
### Prewarming
```Prewarm``` the database is necessary which is the process that loads the tables into the buffer pool. We can simulate the steady-state performance more accurately.
 ```bash
 $ sysbench oltp_read_only prewarm --tables=10 --threads=10
 ```
### Run
Run the benchmark with 10k events, with a ratio of 2 threads per table
 ```bash
 $ sysbench oltp_read_only run --tables=10 --threads=20 --events=10000 --time=0
 ```
 
### Testing Result
 ```bash
 Running the test with following options:
Number of threads: 20
Initializing random number generator from current time


Initializing worker threads...

Threads started!

SQL statistics:
    queries performed:
        read:                            140000
        write:                           0
        other:                           20000
        total:                           160000
    transactions:                        10000  (1077.22 per sec.)
    queries:                             160000 (17235.59 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          9.2813s
    total number of events:              10000

Latency (ms):
         min:                                    4.37
         avg:                                   18.54
         max:                                   69.91
         95th percentile:                       29.19
         sum:                               185396.12

Threads fairness:
    events (avg/stddev):           500.0000/4.46
    execution time (avg/stddev):   9.2698/0.01
 ```
 
You should be awared that sysbench runs the following statements **per event** by default:
- 10x point selects: ```select c from table where id=i```
- 1x simple range: ```select c from table where id between a and b```
- 1x sum range: ```select sum(k) from table where id between a and b```
- 1 x order range: ```select c from table where id between a and b order by c```
- 1 x distinct range: ```select distinct c from table where id between a and b order by c```

### Delete
Delete the tables
```bash
sysbench oltp_read_only cleanup --tables=20
```
## OLTP Write Only

### Prepare 
10 tables, each with 10k rows:
```bash
$ sysbench oltp_write_only prepare --tables=10
```

### Prewarming
```Prewarm``` the database is necessary which is the process that loads the tables into the buffer pool. We can simulate the steady-state performance more accurately.
 ```bash
 $ sysbench oltp_write_only prewarm --tables=10 --threads=10
 ```
### Run
Run the benchmark with 10k events, with a ratio of 2 threads per table
 ```bash
 $ sysbench oltp_write_only run --tables=10 --threads=20 --events=0 --time=30
```

### Testing Result
```bash
Running the test with following options:
Number of threads: 20
Initializing random number generator from current time


Initializing worker threads...

Threads started!

SQL statistics:
    queries performed:
        read:                            0
        write:                           341181
        other:                           170918
        total:                           512099
    transactions:                        85123  (2836.18 per sec.)
    queries:                             512099 (17062.42 per sec.)
    ignored errors:                      672    (22.39 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          30.0117s
    total number of events:              85123

Latency (ms):
         min:                                    0.53
         avg:                                    7.05
         max:                                   67.17
         95th percentile:                       14.21
         sum:                               599892.48

Threads fairness:
    events (avg/stddev):           4256.1500/49.08
    execution time (avg/stddev):   29.9946/0.00
```
 
Each write_only event consists of:
- 1 x index_updates: ```update table set k~k+1 where id=i```
- 1 x non_index_updates: ```update table set c=? where id=i```
- 1 x delete_inserts: ```delete from table where id~i; insert into table (id, k, c, pad) values (...)```

## OLTP Read Write
This test combines the last two in one package. First it runs the same code as in oltp_read_only and then continues with oltp_write_only. We can even reuse the table from the previous benchmark.

Experiment Settings
- ```--range_selects```: Enable/disable all range SELECT queries
- ```--range_size```: Range size for range SELECT queries 
- ```--simple_ranges```Simple range SELECT queries per transaction 
- ```--point_selects``` Point SELECT queries per transaction 
- ```--order_ranges``` SELECT ORDER BY queries per transaction 
- ```--distinct_ranges``` SELECT DISTINCT queries per transaction 
- ```--delete_inserts``` DELETE/INSERT combinations per transaction 
- ```--index_updates``` UPDATE index queries per transaction 
- ```--non_index_updates``` UPDATE non-index queries per transaction 
- ```--sum_ranges``` SELECT SUM() queries per transaction
- ```--skip_trx``` Don’t start explicit transactions (use AUTOCOMMIT)

## Reference
- sysbench’s github: https://github.com/akopytov/sysbench
