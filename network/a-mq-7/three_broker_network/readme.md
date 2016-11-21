### Setup of broker network

create broker1

	./artemis create --name broker1 /workspace/oss/a-mq-migration/network/a-mq-7/three_broker_network/broker1

create broker2

	./artemis create --name broker2 --port-offset 100 /workspace/oss/a-mq-migration/network/a-mq-7/three_broker_network/broker2

create broker3

	./artemis create --name broker1 --port-offset 200 /workspace/oss/a-mq-migration/network/a-mq-7/three_broker_network/broker3

Modify broker1 config

	  <!-- Connectors -->
	    <connectors>
	       <connector name="netty-connector">tcp://localhost:61616</connector>
	    </connectors>


	    <!-- Clustering configuration -->
	    <cluster-connections>
	       <cluster-connection-uri name="my-cluster" address="static://(tcp://localhost:61716)?connectorName=netty-connector;retryInterval=500;messageLoadBalancingType=ON_DEMAND;maxHops=1"/>
	    </cluster-connections>


Modify broker2 config

	  <!-- Connectors -->
	    <connectors>
	       <connector name="netty-connector">tcp://localhost:61716</connector>
	    </connectors>


	    <!-- Clustering configuration -->
	    <cluster-connections>
	       <cluster-connection-uri name="my-cluster" address="static://(tcp://localhost:61616)?connectorName=netty-connector;retryInterval=500;messageLoadBalancingType=ON_DEMAND;maxHops=1"/>
	    </cluster-connections>


modify broker3 config

	  <!-- Connectors -->
	    <connectors>
	       <connector name="netty-connector">tcp://localhost:61816</connector>
	    </connectors>


	    <!-- Clustering configuration -->
	    <cluster-connections>
	       <cluster-connection-uri name="my-cluster" address="static://(tcp://localhost:61616)?connectorName=netty-connector;retryInterval=500;messageLoadBalancingType=ON_DEMAND;maxHops=1"/>
	    </cluster-connections>
		


### Test 1

kill broker 1

start a consumer on broker3

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61816' --user admin --password admin --destination TEST --messageCount 100


send 100 messages to broker2 

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST --messageCount 100

RESULT: consumers on broker3 received messages

WHAT DOES THIS SHOW: The network was configured as follows:

- broker1 connects to broker2, 
- broker2 connects to broker1,  
- broker3 connects to broker1 

broker2 and broker3 are not configured to know about each other, but a connecton is esatablished between broker2 and broker3.  When broker1 is taken away the conneciton between broker2 and broker3 still works


### Test 2
configure broker to explicitly connect to all brokers in network
