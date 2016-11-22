##Setup

Setup broker1 master/slave pair, 
Setup broker2 master/slave pair
Connect statically over the network - connect broker1_master to broker2_master

created broker instances

	./artemis create --name broker1_master /workspace/oss/a-mq-migration/network/a-mq-7/two_master_slave_pairs_network/broker1_master

	./artemis create --name broker1_slave --port-offset 100  /workspace/oss/a-mq-migration/network/a-mq-7/two_master_slave_pairs_network/broker1_slave

	./artemis create --name broker2_master --port-offset 200  /workspace/oss/a-mq-migration/network/a-mq-7/two_master_slave_pairs_network/broker2_master

	./artemis create --name broker2_slave --port-offset 300  /workspace/oss/a-mq-migration/network/a-mq-7/two_master_slave_pairs_network/broker2_slave

###broker1_master

modified config to 


      <paging-directory>../broker1_shared/data/paging</paging-directory>

      <bindings-directory>../broker1_shared/data/bindings</bindings-directory>

      <journal-directory>../broker1_shared/data/journal</journal-directory>

      <large-messages-directory>../broker1_shared/data/large-messages</large-messages-directory>




      <connectors>
         <connector name="netty-connector">tcp://localhost:61616</connector>
         <!-- connector to the slave -->
         <connector name="broker1-slave">tcp://localhost:61716</connector>
		 
		 <!-- connection to broker2 pair-->
		  <connector name="broker2-master">tcp://localhost:61816</connector>
		  
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>ON_DEMAND</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker1-slave</connector-ref>
			   <connector-ref>broker2-master</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>

	  <ha-policy>
	       <shared-store>
	           <master>
	               <failover-on-shutdown>true</failover-on-shutdown>
	           </master>
	        </shared-store>
	  </ha-policy>



###broker1_slave

modifed config as follows

      <paging-directory>../broker1_shared/data/paging</paging-directory>

      <bindings-directory>../broker1_shared/data/bindings</bindings-directory>

      <journal-directory>../broker1_shared/data/journal</journal-directory>

      <large-messages-directory>../broker1_shared/data/large-messages</large-messages-directory>


      <connectors>
         <connector name="netty-connector">tcp://localhost:61716</connector>
         <!-- connector to the broker1 -->
         <connector name="broker1-master">tcp://localhost:61616</connector>
		
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>ON_DEMAND</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker1-master</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>
	  
	  
      <ha-policy>
         <shared-store>
            <slave>
               <failover-on-shutdown>true</failover-on-shutdown>
            </slave>
         </shared-store>
      </ha-policy>
	  
	  
### broker2_master

Modified config as follows

      <paging-directory>../broker2_shared/data/paging</paging-directory>

      <bindings-directory>../broker2_shared/data/bindings</bindings-directory>

      <journal-directory>../broker2_shared/data/journal</journal-directory>

      <large-messages-directory>../broker2_shared/data/large-messages</large-messages-directory>

      <connectors>
         <connector name="netty-connector">tcp://localhost:61816</connector>
         <!-- connector to the slave -->
         <connector name="broker2-slave">tcp://localhost:61916</connector>
		 
		 <!-- connector to broker1 pair-->
		 <connector name="broker1-master">tcp://localhost:61616</connector>
		 
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>ON_DEMAND</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker2-slave</connector-ref>
			   <connector-ref>broker1-master</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>


	  <ha-policy>
	       <shared-store>
	           <master>
	               <failover-on-shutdown>true</failover-on-shutdown>
	           </master>
	        </shared-store>
	  </ha-policy>
	  

### broker2_slave

modified config as follows:

      <paging-directory>../broker2_shared/data/paging</paging-directory>

      <bindings-directory>../broker2_shared/data/bindings</bindings-directory>

      <journal-directory>../broker2_shared/data/journal</journal-directory>

      <large-messages-directory>../broker2_shared/data/large-messages</large-messages-directory>


      <connectors>
         <connector name="netty-connector">tcp://localhost:61916</connector>
         <!-- connector to the broker1 -->
         <connector name="broker2-master">tcp://localhost:61816</connector>
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>ON_DEMAND</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker2-master</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>
	  
	  
      <ha-policy>
         <shared-store>
            <slave>
               <failover-on-shutdown>true</failover-on-shutdown>
            </slave>
         </shared-store>
      </ha-policy>



##Test

###Test 1
AIM: Allow broker2_slave to become live and ensure messages are still coming from broker1_master

start broker1_master

start broker2_master

start broker1_slave

start broker2_slave

kill broker2_master

broker2_slave reports becoming live.

connect to broker2_slave with a consumer

Connect to broker1_master with a producer

RESULT: Messages are pushed through the network from broker1_master to broker2_slave 



###Test 2
AIM: Allow broker1_slave to become live and ensure messages are still coming from broker2_slave (slave active since "Test 1")

kill broker1_master

broker1_slave reports becoming live.

connect to broker1_slave with a consumer

Connect to broker2_slave with a producer

RESULT: Messages are pushed through the network from broker2_slave to broker1_slave 


###Test 3
AIM: if broker1_master and broker2_master are restarted will they become "live" in both pairs and network together

start broker1_master

broker1_master reported becoming live

start broker2_master

broker2_master reported becoming live

connect consumer ot broker2_master

connect producer to broker1_master


RESULT: messages are pushed from broker1_master to consumer on broker2_master.

###Test 4

AIM: push 100,000 message from broker1_master to A-MQ6 consumer configured to failover between broker2_master and broker2_slave


start broker1_master

start broker2_master

start broker1_slave

start broker2_slave

connect to broker2 pair with a failover consumer

	java -jar /opt/a-mq-6/activemq-all.jar  consumer --brokerUrl 'failover:(tcp://localhost:61816,tcp://localhost:61916)' --user admin --password admin --destination TEST --messageCount 100000

start a producer sending messages to broker1_master

	java -jar /opt/a-mq-6/activemq-all.jar  producer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:61916)' --user admin --password admin --destination TEST --messageCount 100000
	

kill broker2_master

RESULT:

- broker2_slave reports becoming live.

- Consumer received 99999 messages 

NOTE: to to rerun later - the 1 missing message may be an issue in the test case.. need to revisit later.




##Questions
