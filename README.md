# BenchmarkingNoSQLDB

Benchmarking NoSQL Databases using YCSB

#### Contributors
- Nitesh Chauhan
- Venciya Antony George
- Sarat Chandra Vysyaraju

## Setup EC2 Instance
1. Login to AWS EC2 console
2. Provision an EC2 instance using publicly available `TSE_SVN_AMI` AWS AMI
![AMI Lookup](https://github.com/niteshch/BenchmarkingNoSQLDB/blob/master/screenshots/TSE-AMI-Provisioning.jpg?raw=true)
![Instance Type Selection](https://github.com/niteshch/BenchmarkingNoSQLDB/blob/master/screenshots/AMI-Instance-type.jpg?raw=true)
3. Follow the default instructions on screen and launch the EC2 instance
4. Once the instance is running, SSH into the EC2 instance
5. Mount the EBS volume at /dev/xvdg to extern folder as stated below. Also, change the ownership of the extern folder
```
sudo mount /dev/xvdg ~/extern/
sudo chown ubuntu:ubuntu ~/extern/
```

At the end of this, you will have an instance with Cassandra 3.5, HBase 1.0.3 and MongoDB 3.2.6 installed in your extern folder.

## Load Data in Database
Now you can proceed to loading data into each of the above databases. You can choose any workload of your choice located inside `~/extern/ycsb-0.8.0/workloads` folder. We have chosen workloada for loading data in to the databases. 
We inserted 1 million records in each database. Each record is of size 1 KB. Hence, the overall size of data load is approximately 1 GB. 

#### Setup Database
Start the database
- Cassandra:
```
cd ~/extern/apache-cassandra-3.5
bin/cassandra
```
- HBase:
```
start-dfs.sh
start-yarn.sh
cd ~/extern/hbase-1.0.3
bin/start-hbase.sh
```
- MongoDB:
```
mongod --dbpath ~/extern/mongodb/data/db/
```

###### Create a table in Cassandra
```
cqlsh> create keyspace ycsb
    WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor': 3 };
cqlsh> USE ycsb;
cqlsh> create table usertable (
    y_id varchar primary key,
    field0 varchar,
    field1 varchar,
    field2 varchar,
    field3 varchar,
    field4 varchar,
    field5 varchar,
    field6 varchar,
    field7 varchar,
    field8 varchar,
    field9 varchar);
```

###### Create a table in HBase
```
hbase(main):001:0> n_splits = 200 # HBase recommends (10 * number of regionservers)
hbase(main):002:0> create 'usertable', 'family', {SPLITS => (1..n_splits).map {|i| "user#{1000+i*(9999-1000)/n_splits}"}}
```

#### Load Data
Once you have ensured that the databases are up and running, you can proceed with loading data in each of the databases. You can use the following commands for reference.
```
cd ~/extern/ycsb-0.8.0/
./bin/ycsb load cassandra2-cql -P workloads/workloada -p recordcount=1000000 -p hosts=localhost

./bin/ycsb load hbase10 -P workloads/workloada -cp /home/ubuntu/extern/hbase-1.0.3/conf -p table=usertable -p columnfamily=family -p recordcount=1000000

./bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://localhost:27017/ycsb?w=0 -p recordcount=1000000
```

## Running a Workload

Once we have loaded the data in the database, we can execute the workload executing the following commands:
```
./bin/ycsb run cassandra2-cql -P workloads/workloada -s -threads 2 -target 40 -p recordcount=1000000 -p hosts=localhost > ~/extern/performance_logs/cassandra/workloada-40.log

./bin/ycsb run hbase10 -P workloads/workloada -s -threads 2 -target 40 -p recordcount=1000000 -cp /home/ubuntu/extern/hbase-1.0.3/conf -p table=usertable -p columnfamily=family > ~/extern/performance_logs/hbase/workloada-40.log

./bin/ycsb run mongodb -P workloads/workloada -s -threads 2 -target 40 -p recordcount=1000000 -p mongodb.url=mongodb://localhost:27017/ycsb?w=0 > ~/extern/performance_logs/mongo/workloada-40.log
```

In this example, we have used in this example we have used the `-threads 2` command line parameter to specify 10 threads, and `-target 40` command line parameter to specify 100 operations per second as the target.

A sample output of the above command on Cassandra database is as follows:
```
Datacenter: datacenter1; Host: localhost/127.0.0.1; Rack: rack1
[OVERALL], RunTime(ms), 27965.0
[OVERALL], Throughput(ops/sec), 35.758984444841765
[READ], Operations, 466.0
[READ], AverageLatency(us), 2243.1180257510728
[READ], MinLatency(us), 715.0
[READ], MaxLatency(us), 31119.0
[READ], 95thPercentileLatency(us), 5359.0
[READ], 99thPercentileLatency(us), 8927.0
[READ], Return=OK, 466
[CLEANUP], Operations, 2.0
[CLEANUP], AverageLatency(us), 1108484.5
[CLEANUP], MinLatency(us), 9.0
[CLEANUP], MaxLatency(us), 2217983.0
[CLEANUP], 95thPercentileLatency(us), 2217983.0
[CLEANUP], 99thPercentileLatency(us), 2217983.0
[UPDATE], Operations, 534.0
[UPDATE], AverageLatency(us), 1257.252808988764
[UPDATE], MinLatency(us), 490.0
[UPDATE], MaxLatency(us), 14007.0
[UPDATE], 95thPercentileLatency(us), 2631.0
[UPDATE], 99thPercentileLatency(us), 5591.0
[UPDATE], Return=OK, 534
```
This output indicates:
- The total execution time was 27965.0 micro-seconds
- The average throughput was 35.75 operations/sec (across all threads)
- There were 466 read operations, with associated average, min, max, 95th and 99th percentile latencies
- There were 534 update operations, with associated average, min, max, 95th and 99th percentile latencies

We ran the experiment for throughput values ranging from 40 to 500 for all the workloads inside the workloads/ folder across all 3 databases as stated in the project report. All the performance logs are committed to `logs/performance_logs/<db_name>` folder of this repository. All the commands used for executing the workloads on the databases are added in the commands.txt file for reference.

You can find the workload files inside the workloads/ folder of this repository.

The description of the workload files is as follows:
- workloada - This file executes 50% read and 50% update operations
- workloadb - This file executes 95% read and 5% update operations
- worloadc - This file executes 100% read operations
- workloadd - This file executes 95% read and 5% insert operations
- customworkloada - This file executes 5% read and 95% update operations

#### Creating a custom workload
One can also choose to create a custom workload. We have added `workload_template` file inside workloads/ folder of this repository. The description of some of the core properties is as follows:

- fieldcount: the number of fields in a record (default: 10)
- fieldlength: the size of each field (default: 100)
- readallfields: should reads read all fields (true) or just one (false) (default: true)
- readproportion: what proportion of operations should be reads (default: 0.95)
- updateproportion: what proportion of operations should be updates (default: 0.05)
- insertproportion: what proportion of operations should be inserts (default: 0)
- scanproportion: what proportion of operations should be scans (default: 0)
- readmodifywriteproportion: what proportion of operations should be read a record, modify it, write it back (default: 0)
- requestdistribution: what distribution should be used to select the records to operate on – uniform, zipfian or latest (default: uniform)
- maxscanlength: for scans, what is the maximum number of records to scan (default: 1000)
- scanlengthdistribution: for scans, what distribution should be used to choose the number of records to scan, for each - scan, between 1 and maxscanlength (default: uniform)
- insertorder: should records be inserted in order by key (“ordered”), or in hashed order (“hashed”) (default: hashed)
- operationcount: number of operations to perform.
- maxexecutiontime: maximum execution time in seconds. The benchmark runs until either the operation count has exhausted or the maximum specified time has elapsed, whichever is earlier.
- table: the name of the table (default: usertable)
- recordcount: number of records to load into the database initially (default: 0)



