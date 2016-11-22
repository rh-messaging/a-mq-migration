##Setup

created broker1_master

	./artemis create --name broker1_master /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-7/cluster-static-discovery/broker1_master

create broker1_slave

	./artemis create --name broker1_slave --port-offset 100  /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-7/cluster-static-discovery/broker1_slave	


modify broker1_master config:

      <paging-directory>../shared_store/data/paging</paging-directory>

      <bindings-directory>../shared_store/data/bindings</bindings-directory>

      <journal-directory>../shared_store/data/journal</journal-directory>

      <large-messages-directory>../shared_store/data/large-messages</large-messages-directory>
	  
...

      <connectors>
         <connector name="netty-connector">tcp://localhost:61616</connector>
         <!-- connector to the slave -->
         <connector name="broker1-slave">tcp://localhost:61716</connector>
      </connectors>
	  
	  
      <cluster-connections>
         <cluster-connection name="my-cluster">
            <address>jms</address>
            <connector-ref>netty-connector</connector-ref>
            <retry-interval>500</retry-interval>
            <use-duplicate-detection>true</use-duplicate-detection>
            <message-load-balancing>STRICT</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker1-slave</connector-ref>
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
	  


modify broker1_slave config

      <paging-directory>../shared_store/data/paging</paging-directory>

      <bindings-directory>../shared_store/data/bindings</bindings-directory>

      <journal-directory>../shared_store/data/journal</journal-directory>

      <large-messages-directory>../shared_store/data/large-messages</large-messages-directory>
	  

...

	  
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
            <message-load-balancing>STRICT</message-load-balancing>
            <max-hops>1</max-hops>
            <static-connectors>
               <connector-ref>broker1-master</connector-ref>
            </static-connectors>
         </cluster-connection>
      </cluster-connections>
	  
	  

##Tests

### Test 1

AIM: ensure master/slave are aware of each other.

started broker1_master

broker1_master reported
	
	10:29:04,917 INFO  [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live

started broker1_slave

broker1_slave reported:

	10:37:24,469 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced

shutdown broker1_master

broker1_slave reported

	10:38:50,753 WARN  [org.apache.activemq.artemis.core.client] AMQ212037: Connection failure has been detected: AMQ119015: The connection was disconnected because of server shutdown [code=DISCONNECTED]
	10:38:51,167 WARN  [org.apache.activemq.artemis.core.client] AMQ212004: Failed to connect to server.
	10:38:51,483 INFO  [org.apache.activemq.artemis.core.server] AMQ221003: Deploying queue jms.queue.DLQ
	10:38:51,496 INFO  [org.apache.activemq.artemis.core.server] AMQ221003: Deploying queue jms.queue.ExpiryQueue
	10:38:51,688 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started Acceptor at 0.0.0.0:61716 for protocols [CORE,MQTT,AMQP,HORNETQ,STOMP,OPENWIRE]
	...
	10:38:51,697 INFO  [org.apache.activemq.artemis.core.server] AMQ221010: Backup Server is now live
	
RESULT: slave took over as expected

### Test 2

AIM: Does slave return to backup status when master broker is restartd.

started broker1_master

started broker1_slave

shutdown broker1_master

broker1_slave became "live"

restart broker1_master 

broker_slave reported the following:

	10:49:29,471 INFO  [org.apache.activemq.artemis.core.server] AMQ221008: live server wants to restart, restarting server in backup
	INFO  | Thread-12 | Destroyed dispatch-hawtio-console plugin
	INFO  | Thread-12 | Destroyed artemis-plugin plugin
	10:49:29,935 INFO  [org.apache.activemq.artemis.core.server] AMQ221002: Apache ActiveMQ Artemis Message Broker version 1.3.0.amq-700005-redhat-1 [1bd89364-b096-11e6-b2ce-3c07540d37d0] stopped, uptime 4 minutes
	10:49:31,936 INFO  [org.apache.activemq.artemis.core.server] AMQ221000: backup Message Broker is starting with configuration Broker Configuration (clustered=true,journalDirectory=../shared_store/data/journal,bindingsDirectory=../shared_store/data/bindings,largeMessagesDirectory=../shared_store/data/large-messages,pagingDirectory=../shared_store/data/paging)
	10:49:31,936 INFO  [org.apache.activemq.artemis.core.server] AMQ221032: Waiting to become backup node
	10:49:31,937 INFO  [org.apache.activemq.artemis.core.server] AMQ221033: ** got backup lock
	10:49:31,937 INFO  [org.apache.activemq.artemis.core.server] AMQ221013: Using NIO Journal
	10:49:31,937 WARN  [org.apache.activemq.artemis.core.server] AMQ222007: Security risk! Apache ActiveMQ Artemis is running with the default cluster admin user and default password. Please see the cluster chapter in the ActiveMQ Artemis User Guide for instructions on how to change this.
	10:49:31,940 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-server]. Adding protocol support for: CORE
	10:49:32,126 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-amqp-protocol]. Adding protocol support for: AMQP
	10:49:32,126 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-hornetq-protocol]. Adding protocol support for: HORNETQ
	10:49:32,127 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-mqtt-protocol]. Adding protocol support for: MQTT
	10:49:32,127 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-openwire-protocol]. Adding protocol support for: OPENWIRE
	10:49:32,127 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-stomp-protocol]. Adding protocol support for: STOMP
	10:49:32,136 WARN  [org.apache.activemq.artemis.core.server] AMQ222040: Server is stopped
	10:49:32,137 INFO  [org.apache.activemq.artemis.core.server] AMQ221109: Apache ActiveMQ Artemis Backup Server version 1.3.0.amq-700005-redhat-1 [1bd89364-b096-11e6-b2ce-3c07540d37d0] started, waiting live to fail before it gets active
	10:49:32,141 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced

broker1_master reported 

	10:49:29,086 INFO  [org.apache.activemq.artemis.core.server] AMQ221034: Waiting indefinitely to obtain live lock
	10:49:29,258 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced
	10:49:29,517 WARN  [org.apache.activemq.artemis.core.client] AMQ212037: Connection failure has been detected: AMQ119015: The connection was disconnected because of server shutdown [code=DISCONNECTED]
	10:49:29,517 WARN  [org.apache.activemq.artemis.core.client] AMQ212037: Connection failure has been detected: AMQ119015: The connection was disconnected because of server shutdown [code=DISCONNECTED]
	10:49:29,590 INFO  [org.apache.activemq.artemis.core.server] AMQ221035: Live Server Obtained live lock
	10:49:30,101 INFO  [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live


RESULT: Broker1_slave detected broker1_master restart and went back to slave. Broker1_master became active.
