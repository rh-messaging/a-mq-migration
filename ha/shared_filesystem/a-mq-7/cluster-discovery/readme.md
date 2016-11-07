
# WORKING DOCUMENT: THIS NEEDS TO BE REVIEWED -

##How to run


##How was it Setup:

To create broker1

./artemis create  --shared-store /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-7/cluster-discovery/broker1

To create broker2

./artemis create  --shared-store --slave /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-7/cluster-discovery/broker2

specify user/password as admin/admin when creating brokers


edited the broker.xml of each broker to point at the same shared store

## observations:

- sets up a broadcast/discovery network
- it seems to announce back across the network. 
- only when it cannot contact its master does it become the master



## Questions: 
- is it mandiatory to have a  cluster defined do do master/slave with shared storage
- does it use a filesystem lock as well or will it just reply on the cluster 
- what happens if there is a physical network failure between broker1 and broker2

##Notes: 
- failover-on-shutdown - need to highlight as that is different behaviour to A-MQ 6 share storage, master/slave


##Tests run:

### Test 1:

Objective: Does broker1 (master) and broker2 (slave)

send one hundread message to broker1 (master)

 java -jar /opt/a-mq-6/activemq-all.jar  producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST
 
kill -9 broker1

broker2 appeared to detect broker1 was no longer available 

attach consumer to slave 

java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST

Consumer seemed to consume all the sent message - it proves it is using the same store.

###Test 2:

Objective: Does the original master take control of the lock

Restart broker1 (original master)

Broker2 appeared to gives up the lock and gets backup lock

Broker1 starts as master


###Test 3:

Objective: To see if A-MQ6 client with failover use the "network topology" info - reconnect to slave broker when it goes live

Attach consumer to broker1, 

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'failover:(tcp://localhost:61616)' --user admin --password admin --destination TEST

killed the master - broker 1

produced message to the slave 

	java -jar /opt/a-mq-6/activemq-all.jar  producer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST

Consumer did not seem to receive the messages

**NOTE** when broker1 was restart, got the lock and went live, the consumer did reconnect to this broker and did consume the messages


###Test 4:

Objective: To see if A-MQ6 client with failover using both brokers defined in the failover url - reconnect to slave broker when it goes live

Attach consumer to broker 1

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:61716)' --user admin --password admin --destination TEST

kill broker1 

Consumer seemed to successfully connect to broker2

Produced messages using the failover url below

	java -jar /opt/a-mq-6/activemq-all.jar  producer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:61716)' --user admin --password admin --destination TEST

Consumer consumed the messages













 