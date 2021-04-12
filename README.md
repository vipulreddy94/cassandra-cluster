# cassandra-cluster
Setting up a 3 node cassandra cluster, spread across two datacenters, using docker.

1. Download cassandra and set up a 3 node cluster spread across 2 data centers (nodes cas1 and cas2 in DC1 while node cas3 in DC2) 
  - Set memory in Docker Preferences to 5 GB, as each node might take up around 1.5 GB
  - Pull the Cassandra docker image from the docker registry --> docker pull cassandra. 
  - Run the first container (which is the first node cas1). Name of the cluster is MyCluster and GossipingProtocolFileSnitch is used for communication between          nodes. (-p flag is used to expose ports for networking, and -e flag is used for setting environment variables on the container). 

     container 1
     
     ```
     docker run --name cas1 -p 9042:9042 -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter1 -d cassandra
     ```
  - Run the second and third containers cas2 (datacenter1) and cas3 (datacenter2) with the same clustername MyCluster. In order to form a cluster and communicate, containers have to be connected - containers 2 and 3 are fed the IP address of the container 1 through an environment variable called CASSANDRA_SEEDS.
     
     container 2
     
     ```
     docker run --name cas2 -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cas1)" -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter1 -d cassandra
     ```
     
     container 3
     
     ```
     docker run --name cas3 -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cas1)" -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter2 -d cassandra
     ```     
  
  - After, a while run the below command to check the status of the cluster
    ```
    docker exec -ti cas1 nodetool status
    ```
    
  - Since the cluster is now set up, we can create keyspaces and tables in the cluster, through the CassandraQuery Language. We can input CQL queries through the cqlsh. We can access this on any of the nodes.
    ```
    docker exec -ti cas1 cqlsh
    ```
    Once, on the cqlsh, create a namespace with a replication factor of 3 and class= NetworkTopologyStrategy. 
    ```
    CREATE KEYSPACE IF NOT EXISTS testkeypsace with REPLICATION = {'class':'NetworkTopologyStrategy', 'datacenter1' :1, 'datacenter2' : 1}
    ```
    To create a table, use the below command
    ```
    CREATE TABLE testkeyspace.testtable (int id primary key, name text);
    INSERT INTO testkeyspace.testtable(1,'Robert');
    INSERT INTO testkeyspace.testtable(2,'Rob');
    INSERT INTO testkeyspace.testtable(3,'Roberto');
    SELECT name from testkeyspace.testtable where id = 2;
   
    ```
    
    To check the status of table and where its stored, use the below command.
    
    ```
    docker exec -ti cas1 nodetool status
    ```
    
    
 Sample![image](https://user-images.githubusercontent.com/47663871/114228684-d082f000-9944-11eb-9a65-619641faea48.png)



  - Reference 1 : https://medium.com/@mertcal/running-cassandra-cluster-on-docker-d9a44aafebb9
  - Reference 2 : https://blog.toadworld.com/2018/02/13/build-a-cassandra-cluster-on-docker 
  - Reference 3 : https://towardsdatascience.com/getting-started-with-apache-cassandra-and-python-81e00ccf17c9
