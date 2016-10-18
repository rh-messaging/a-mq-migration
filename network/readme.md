## Network

### A-MQ 6

Two brokers. One in a-mq-6/local/

        <transportConnectors>
            <transportConnector name="openwire" uri="tcp://0.0.0.0:0?maximumConnections=1000"/>
        </transportConnectors>

        <networkConnectors>
            <networkConnector uri="static:(tcp://localhost:61617)" userName="admin" password="admin"/>
        </networkConnectors> 

The other in a-mq-7/remote

        <transportConnectors>
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61617?maximumConnections=1000"/>
        </transportConnectors>

        <networkConnectors>
            <networkConnector uri="static:(tcp://localhost:61616)" userName="admin" password="admin"/>
        </networkConnectors>          


Test

Start two brokers

	cd a-mq-6/local
	bin/start

and

	cd a-mq-6/remote
	bin/start


Subscribe to one broker

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61617' --user admin --password admin --destination TEST

Produce to other

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST


### A-MQ 7

Two brokers. One in a-mq-7/local

      <acceptors>
         <acceptor name="artemis">tcp://0.0.0.0:61616?tcpSendBufferSize=1048576;tcpReceiveBufferSize=1048576</acceptor>
      </acceptors>

      <connectors>
         <connector name="remote-connector">tcp://localhost:61617</connector>
      </connectors>

      <queues>
         <queue name="jms.queue.TEST">
            <address>jms.queue.TEST</address>
         </queue>
      </queues>        

      <bridges>
         <bridge name="bridge">
            <queue-name>jms.queue.TEST</queue-name>
            <forwarding-address>jms.queue.TEST</forwarding-address>
            <user>admin</user>
            <password>admin</password>
            <static-connectors>
               <connector-ref>remote-connector</connector-ref>
            </static-connectors>
         </bridge>
      </bridges>

The other in a-mq-7/remote

      <acceptors>
         <acceptor name="artemis">tcp://0.0.0.0:61617?tcpSendBufferSize=1048576;tcpReceiveBufferSize=1048576</acceptor>
      </acceptors>

      <connectors>
         <connector name="remote-connector">tcp://localhost:61616</connector>
      </connectors>

      <queues>
         <queue name="jms.queue.TEST">
            <address>jms.queue.TEST</address>
         </queue>
      </queues>      

      <bridges>
         <bridge name="bridge">
            <queue-name>jms.queue.TEST</queue-name>
            <forwarding-address>jms.queue.TEST</forwarding-address>
            <user>admin</user>
            <password>admin</password>            
            <static-connectors>
               <connector-ref>remote-connector</connector-ref>
            </static-connectors>
         </bridge>
      </bridges>

Test

Start two brokers

	cd a-mq-7/local
	bin/artemis run

and

	cd a-mq-7/remote
	bin/artemis run


Subscribe to one broker

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'tcp://localhost:61617' --user admin --password admin --destination TEST

Produce to other

	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'tcp://localhost:61616' --user admin --password admin --destination TEST      
