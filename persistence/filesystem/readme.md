#Working documents - needs review. 

##A-MQ 6 kahadb

Creates a broker using kahadb. Uses many of the available configuration options for kahadb.

	export KARAF_ETC=/workspace/oss/a-mq-migration/persistence/filesystem/a-mq-6/etc
	export KARAF_DATA=/workspace/oss/a-mq-migration/persistence/filesystem/a-mq-6/data
	mkdir /workspace/oss/a-mq-migration/persistence/filesystem/a-mq-6/data
	cd /opt/a-mq-6/
	bin/amq
 
start a consumer

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST

start a producer

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST 

Consumer receives 1000 message - OK



## A-MQ 7 NIO Journal


### Setup

./artemis create --nio /workspace/oss/a-mq-migration/persistence/filesystem/a-mq-7/nio_broker

### Run

To run the a-mq-7 broker

	cd a-mq-7/nio_broker/bin
	./artemis run

### Test

#### Test 1

Objective: basic functionality - ensure messages are persisted between restarts

Start producer 

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST

kill the broker

	kill -9 <BROKER_PID>
	
Start the broker 

	./artemis run

Start consumer

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST

Ensure all messages are consumed

Result: All messages where consumed

#### Test 2

Objective: Ensure journal files are removed when not being used. Understand how journal files are being controled by configuration


Start producer with many large messages


	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST  --messageSize 100000 --messageCount 3010

When producer is finished, check journal directory/number of file size

Start consumer to consumer **most** of the messages

	 java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST --messageCount 3000

When consumer is finished, check journal directory/number of file size


Start consumer to consume last message from the destination

	 java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST --messageCount 10

When consumer is finished, check journal directory/number of file size


Result:

1) Observed that store journals only grew to approx 110MB. 
Then the PAGING store appeared to grow - see Observation 1.

2) With only ten messages in the broker the journal still seem to the 110 MB of disk used. The page directory appeared to only have one page left

TODO Find the configuration that influces how many/how often store files are compacted/discarded. 
**NOTE** I suspect there is a pooling journal mechanism/per allocation in play but need to research

3) Even after all the messages have been consumd the file have not been reduced 


TODO: What configuration is influencing number of journal files kept


#### Test 3

TODO

Objective: how to use the artemis "data" tools. Print the journal

Push 200 messagee into dest



##Observations:

####1. Paging 
Question: I noticed that  when PAGING kicked in, the NIO store did *not* grow but the paging store grew.  I kind of expected the messages to be written to the store and  before being paged out to disk.  Does the paging mechanism occur before the message are placed in the store?
ATaylor: the messages are actually in the page store not the journal, only the reference is in the journal.  The page is written first then the reference to the journal and then a sync happens. If the page is updated but the reference fails the messsgae is regarded as not delivered and ignored after any reload/failure. 

Question: When more space becomes available in the journal store (after messages are consumed from the destination) will those paged message get pulled in from the page file and written to the journal store?
ATaylor: Nope, just read into memory and  the reference is acked when consumed and the page record deleted. Its just references that are important, messages can be in the journal, in paging or in a large message store


##Questions:

- does A-MQ7 offer something similar to mkahadb
- what file systems (particularly network shared filesystems) is A-MQ7 supported on  eg NFSv4, CIF, etc??

 

##TODO

- review all A-MQ6 configuration attributes . Identify if they are applicable OR not in A-MQ 7. For those that are applicable highlight under observation section. For those that are not highlight why not under observation section 

