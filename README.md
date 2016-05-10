# BenchmarkingNoSQLDB

Benchmarking NoSQL Databases using YCSB

#### Contributors
- Nitesh Chauhan
- Venciya Antony George
- Sarat Chandra Vysyaraju

## Setup
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

## Loading Data in Database
Now you can proceed to loading data into each of the above databases. You can choose any workload of your choice located inside `~/extern/ycsb-0.8.0/workloads` folder. We have chosen workloada for loading data in to the databases. 
We inserted 1 million records in each database. Each record is of size 1 KB. Hence, the overall size of data load is approximately 1 GB. 

Make sure you start the database before trying to load data in any of the databases. 
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

Once you have ensured that the databases are up and running, you can proceed with loading data in each of the databases. You can use the following commands for reference.
```
cd ~/extern/ycsb-0.8.0/
./bin/ycsb load cassandra2-cql -P workloads/workloada -p recordcount=1000000 -p hosts=localhost

./bin/ycsb load hbase10 -P workloads/workloada -cp /home/ubuntu/extern/hbase-1.0.3/conf -p table=usertable -p columnfamily=family -p recordcount=1000000

./bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://localhost:27017/ycsb?w=0 -p recordcount=1000000
```




