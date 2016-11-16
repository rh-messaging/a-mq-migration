
### Steps to create broker network

create broker1

	./artemis create --name broker1 /workspace/oss/a-mq-migration/network/a-mq-7/clustered-static-discovery-uri/broker1	
	
	username admin
	password admin
	role admin

create broker2

	./artemis create --name broker2 --port-offset 100 /workspace/oss/a-mq-migration/network/a-mq-7/clustered-static-discovery-uri/broker2

	username admin
	password admin
	role admin


add broker network configuration based on 

/opt/a-mq-7/examples/features/clustered/clustered-static-discovery-uri/src/main/resources/activemq/server1

in broker1 

      <!-- Connectors -->
      <connectors>
         <connector name="netty-connector">tcp://localhost:61616</connector>
      </connectors>
	  
	  
      <!-- Clustering configuration -->
      <cluster-connections>
         <cluster-connection-uri name="my-cluster" address="static://(tcp://localhost:61716)?connectorName=netty-connector;retryInterval=500;messageLoadBalancingType=STRICT;maxHops=1"/>
      </cluster-connections>
	  
	  
in broker2

      <!-- Connectors -->
      <connectors>
         <connector name="netty-connector">tcp://localhost:61716</connector>
      </connectors>
	  
	  
      <!-- Clustering configuration -->
      <cluster-connections>
         <cluster-connection-uri name="my-cluster" address="static://(tcp://localhost:61616)?connectorName=netty-connector;retryInterval=500;messageLoadBalancingType=STRICT;maxHops=1"/>
      </cluster-connections>

Please note

	Had to add  <queue name="TEST"/>  to the   <jms xmlns="urn:activemq:jms">. It did not seem to work otherwise 


### Test 1

Send 1000 messages to broker1

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST

consumer 500 message from broker2

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST

consume 500 messages from broker1 
	
	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST


Result: Messages where consumed as expected and 500 available on each broker.

NOTE: messages will not be available anymore in broker1   


### Test 2

- changed messageLoadBalancingType=ON_DEMAND
- added <redistribution-delay>0</redistribution-delay>

Send 1000 messages to broker1

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST

Consumed 100 messages from broker2
   
	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST --messageCount 100
   

Consumed 100 messages from broker1
    
	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST --messageCount 100


Consumed 100 messages from broker2
    
	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST --messageCount 100


result: messages consumed from both brokers, interleaved. 700 messages let sitting on broker2 (this is the last broker consumerd rome) 


Note: observed in the JMX that ALL the remaining messages would be pushed across the network to the broker with the consumer. This resulted in a messages travelling back and forth when a consumer would attach and then disconnect,  


####NOTES

- according to docs redistribution delay only works when messageLoadBalancingType=ON_DEMAND. The redistribution will only take effect when no consumers are available on the queue.  This is similiar to the A-MQ6 replaywhennoconsumer allowing the messages to go back to the original broker if no consumers. However in A-MQ 6 it there are consumers 
on another a different broker(not source broker) they will get some of the message. If the A-MQ7 docs are correct they will not get redistributed in the network until no consumers - this is a slight difference in message distribution.
 
 



#### TODO

- find out recommendations for maxhops and redistribution values
- is there a way to throttle/limit the number of messages passed between the brokers in the network.
