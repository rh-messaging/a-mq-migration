
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


#### TODO

- look at DEMAND strategy
- look at REDISTRIBUTION approach
- find out recommendations for maxhops and redistribution values
