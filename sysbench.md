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

## Sysbench Data Load

- load:
```bash
sysbench --test=/home/lbh/sysbench-1.0.12/sysbench/tests/oltp_legacy/oltp.lua --mysql-host=localhost  --mysql-db=sbtest --mysql-user=root --mysql-password=evia6587 --max-requests=0  --oltp-table-size=2000000 --max-time=600  --oltp-tables-count=200 --report-interval=10 --db-ps-mode=disable  --random-points=10 --mysql-table-engine=InnoDB --mysql-port=3306 --num-threads=128  prepare
```

## Sysbench Testing
- run:
```bash
sysbench --test=/home/lbh/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua --mysql-host=localhost  --mysql-db=sbtest --mysql-user=root --mysql-password=evia6587 --max-requests=0  --oltp-table-size=2000000 --time=259200  --oltp-tables-count=200 --report-interval=10 --db-ps-mode=disable  --random-points=10 --mysql-table-engine=InnoDB --mysql-port=3307 --mysql-socket=/tmp/mysql.sock1 --num-threads=128  run |tee /home/lbh/result/sb-ns.txt
```
