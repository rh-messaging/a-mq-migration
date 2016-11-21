## Setup

Based on /opt/a-mq-7/examples/features/clustered
NOTE: I could not get "allow-direct-connections-only" working with cluster-connection-uri. Perhaps this was an issue with my setup - not sure.



create broker1 

	./artemis create --name broker1 /workspace/oss/a-mq-migration/network/a-mq-7/linear_broker_network/broker1

create broker2

	./artemis create --name broker2 --port-offset 100 /workspace/oss/a-mq-migration/network/a-mq-7/linear_broker_network/broker2

create broker3

	 ./artemis create --name broker3 --port-offset 200 /workspace/oss/a-mq-migration/network/a-mq-7/linear_broker_network/broker3

modify broker1 config to include TEST in the existing urn:activemq:jms

    <jms xmlns="urn:activemq:jms">
       <queue name="DLQ"/>
       <queue name="ExpiryQueue"/>
 	   <queue name="TEST"/>
    </jms>

AND

      <connectors>
         <connector name="netty-connector">tcp://localhost:61616</connector>
         <!-- connector to the server1 -->
         <connector name="broker2-connector">tcp://localhost:61716</connector>
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>STRICT</message-load-balancing>
            <max-hops>2</max-hops>
            <static-connectors allow-direct-connections-only="true">
               <connector-ref>broker2-connector</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>

modify broker2 config to include TEST in the existing urn:activemq:jms

    <jms xmlns="urn:activemq:jms">
       <queue name="DLQ"/>
       <queue name="ExpiryQueue"/>
 	   <queue name="TEST"/>
    </jms>

AND

      <connectors>
         <connector name="netty-connector">tcp://localhost:61716</connector>
         <!-- connector to the server1 -->
         <connector name="broker3-connector">tcp://localhost:61816</connector>
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>STRICT</message-load-balancing>
            <max-hops>2</max-hops>
            <static-connectors allow-direct-connections-only="true">
               <connector-ref>broker3-connector</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>
	  	  

modify broker3 config to include TEST in the existing urn:activemq:jms

    <jms xmlns="urn:activemq:jms">
       <queue name="DLQ"/>
       <queue name="ExpiryQueue"/>
 	   <queue name="TEST"/>
    </jms>

AND


      <!-- Connectors -->
      <connectors>
         <connector name="netty-connector">tcp://localhost:61816</connector>
      </connectors>

      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>STRICT</message-load-balancing>
            <max-hops>2</max-hops>
         </cluster-connection>
      </cluster-connections>
	  
##Test

### Test 1
AIM: ensure messages are flowing from broker1 to broker3

send 100 messages to broker1

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST --messageCount 100

RESULT:

33 messages were sent to  TEST on broker2 and broker3
34 messages were sent to  TEST on broker1 (local)

### Test 2
AIM: do messages flow from broker3 to broker1

send 100 messages to broker3

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61816' --user admin --password admin --destination TEST --messageCount 100

RESULT:

All 100 messages were sent to  TEST on broker3 (local). No message were dispatched accross network


### Test 3
AIM: introduce messages to broker2 will they flow to broker3?

send 100 messages to broker2

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61716' --user admin --password admin --destination TEST --messageCount 100

RESULT:

50 messages were sent to  TEST on broker2 (local).
50 messages were sent to  TEST on broker3 .
no messages dispatch to broker1


##Observations

- even in a linear network messages where distrbuted evenly between broker when using STRICT


##Questions



