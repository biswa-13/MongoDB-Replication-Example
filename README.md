# MongoDB-Replication-Example
This is a step by step guide to setup the mongoDB replication for high availability.
Author@ BiswaRanjan Samal
Email@ bichhubiswa@gmail.com
------------------------------------------------------------------------------------
Steps for creating 3 node ReplicaSet in your local Machine :-

0 -> First of all shutdown the running mongod and mongo instances .

1 -> Creating/Modefying your mongod.conf file
systemLog:
   destination: file
   path: "C:/Program Files/MongoDB 2.6 Standard/bin/data/log/mongo.log"
   logAppend: true
storage:
   dbPath: "C:/Program Files/MongoDB 2.6 Standard/bin/data/db"
   journal:
      enabled: true
net:
   bindIp: 127.0.0.1, LazyEasyBRS
   port: 27017
security:
   authorization: "disabled"
replication:
   replSetName: "pzeonRS"

Note: 
1.1: Above configuration files should be copied and 3, *.conf file should be created with diffrent name for runnuing 3 mongod instance.
1.2: From *.conf file these {systemLog.path, storage.dbPath, net.port} key values should be changed and the values hsould be unique .

2 -> Running your mongod instance

syntax: mongod --config "pathToYourMongod.conf"
Ex:     mongod --config "mongod.conf" 
// my mongod.conf & monogd is present at C:\Program Files\MongoDB 2.6 Standard\bin

3 -> Open mongo and instatiate the ReplicaSet

3.1: syntax: mongo --port 27017
3.2: syntax: rs.initiate() // this will instatiate the replicaSet
3.3: syntax: rs.status() // to see the status of the replicaEet 
3.4: syntax: rs.config() // to see the configuration of the replicaSet, it should show only one object 

4 -> Add members to the replicaSet
4.1: Repeat step-2 for running other two mongod instances with their respective mongod.conf file.
	 Ex: mongod --config "mongod1.conf"
	 Ex: mongod --config "mongod2.conf"
4.2: Now goto the mongo instance that is opened in step-3, and from here we will add the other two mongod instance to our replicaSet
	syntax: rs.add("hostName:port")
	ex:   : rs.add("LazyEasyBRS:27018")
	ex:   : rs.add("LazyEasyBRS:27019")

4.3: Now check the status of the replicaSet it should show 3 objects
	syntax: rs.status()
	output:
	{
        "set" : "pzeonRS",
        "date" : ISODate("2018-05-17T11:32:29Z"),
        "myState" : 1,
        "members" : [
                {
                        "_id" : 0,
                        "name" : "LazyEasyBRS:27017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 4509,
                        "optime" : Timestamp(1526553010, 2046),
                        "optimeDate" : ISODate("2018-05-17T10:30:10Z"),
                        "electionTime" : Timestamp(1526552248, 1),
                        "electionDate" : ISODate("2018-05-17T10:17:28Z"),
                        "self" : true
                },
                {
                        "_id" : 1,
                        "name" : "LazyEasyBRS:27018",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 4327,
                        "optime" : Timestamp(1526553010, 2046),
                        "optimeDate" : ISODate("2018-05-17T10:30:10Z"),
                        "lastHeartbeat" : ISODate("2018-05-17T11:32:28Z"),
                        "lastHeartbeatRecv" : ISODate("2018-05-17T11:32:27Z"),
                        "pingMs" : 0,
                        "syncingTo" : "LazyEasyBRS:27017"
                },
                {
                        "_id" : 2,
                        "name" : "LazyEasyBRS:27019",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 4315,
                        "optime" : Timestamp(1526553010, 2046),
                        "optimeDate" : ISODate("2018-05-17T10:30:10Z"),
                        "lastHeartbeat" : ISODate("2018-05-17T11:32:27Z"),
                        "lastHeartbeatRecv" : ISODate("2018-05-17T11:32:28Z"),
                        "pingMs" : 0,
                        "syncingTo" : "LazyEasyBRS:27017"
                }
        ],
        "ok" : 1
	}

5 -> Configuring read preference mode to secondary

5.1: syntax: print(db.getMongo().getReadPrefMode()); // to get the current readPreference mode
5.2: syntax: db.getMongo().setReadPref("secondary"); // to set the readPreference to secondary
5.3: open any of the secondary node's mongo 
	 syntax : mongo --port 27018
	 then ruin the below command for acknoweldgement
	 syntax: rs.slaveOk()
6 -> confirming read query execution in secondary node

6.1: lets run mongotop for all the instances to track the amount of time a MongoDB instance spends reading and writing data. 
	 syntax: mongotop -h ip:port seconds  // seconds: int value to refesh the dashboard
	 ex:     mongotop -h 127.0.0.1:27017 5 	
	 ex:     mongotop -h 127.0.0.1:27018 5
	 ex:     mongotop -h 127.0.0.1:27019 5

6.2: lets execute a count command on mongo with port 27018 and look at the mongotop of the same port 27018

	 ex: db.thread.count({bundleId:{$gt:124}}) //query for nurseNav db

Note: Below commands are considered as the read commands. (source : https://docs.mongodb.com/v2.6/reference/read-preference/#secondary)

group
mapReduce 
aggregate 
collStats
dbStats
count
distinct
geoNear
geoSearch
geoWalk
parallelCollectionScan

--------------------------------Optional Steps-----------------------------------------
7 -> Adding Arbiter to the ReplicaSet
7.1: Create a data directory (e.g. dbPath) for the arbiter.
     syntax: mkdir /data/arb
7.2: start/run the arbiter
     syntax: mongod --port 30000 --dbpath data/arb --replSet pzeonRS
7.3: Add the arbiter to the replica set
     syntax: rs.addArb("LazyEasyBRS:30000")

8 -> Editing mongorc.js 
     This file is used to execute startup scripts while mongo shell gets started,
     we will set rs.slaveOk() command so that the replica set will always tries to read data from secondary.
     
     // reference: https://www.youtube.com/watch?v=g6D_CTX5WI0
     // refrence: https://docs.mongodb.com/v2.6/reference/program/mongo/
8.1: Loacte the mongorc.js file
     syntax:C:/Users/UserName/mongorc.js
     ex:    C:/Users/HP/mongorc.js
8.2: Add the following command in the file and save it
     syntax: rs.slaveOk()

9 -> Setting a hidden node to be used exclusively for analytics purpose
9.1: Follow step 1 & 4 and run another server in a new port, or you can also convert a 
     exisring secondary node to hidden node.
     syntax: cfg = rs.conf()
             cfg.members[4].priority = 0 // members[4] represents the index of the node in the replicaset
             cfg.members[4].hidden = true
             rs.reconfig(cfg)

     
References: 
-> How the election happens in MongoDB Replication
// https://docs.mongodb.com/v2.6/core/replica-set-elections/#connections

-> to view the servers query execution status
// https://stackoverflow.com/questions/34267909/how-to-check-application-is-reading-from-secondary-node-in-mongodb

-> detailed replicaset example
// http://igorbicanic.blogspot.in/2014/11/install-and-configure-mongodb-replica.html

-> blogs to guide replicaSet configurations
// https://severalnines.com/blog/become-mongodb-dba-basics-mongodb-configuration
// https://severalnines.com/blog/become-mongodb-dba-how-scale-reads

-> Setting Hidden Node
//https://docs.mongodb.com/v2.6/core/replica-set-hidden-member/

-> Mongodb all configuration files
//https://github.com/mongodb/docs/tree/master/source/tutorial

