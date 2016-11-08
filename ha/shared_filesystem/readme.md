##A-MQ 6 Kahadb


Start two brokers, broker1 and broker2. Point both broker at same kahadb directory.


		<persistenceAdapter>
		  <kahaDB directory="${shared-kahadb}" lockKeepAlivePeriod="5000">
		    <locker>
			  <shared-file-locker lockAcquireSleepInterval="10000"/>
		      </locker>
		  </kahaDB>
		</persistenceAdapter>

Start broker1


	export KARAF_ETC=/workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker1/etc
	export KARAF_DATA=/workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker1/data
	mkdir /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker1/data
	cd /opt/a-mq-6/
	bin/amq

Start broker2

	export KARAF_ETC=/workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker2/etc
	export KARAF_DATA=/workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker2/data
	mkdir /workspace/oss/a-mq-migration/ha/shared_filesystem/a-mq-6/broker2/data
	cd /opt/a-mq-6_dist2/
	bin/amq
	
broker1 will start as master and broker2 will be slave - it is waiting to get lock on kahadb

Start a consumer 

	java -jar /opt/a-mq-6/activemq-all.jar consumer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:61617)' --user admin --password admin --destination TEST


Start a producer and send one message

    java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:61617)' --user admin --password admin --destination TEST --messageCount 1

kill broker1

	kill <_broker1_PID__>


Wait 10 seconds and start a producer and send one message. Broker2 should have become master.
   
	java -jar /opt/a-mq-6/activemq-all.jar producer --brokerUrl 'failover:(tcp://localhost:61616,tcp://localhost:616167)' --user admin --password admin --destination TEST --messageCount 1

Second message should reach consumer - OK



##TODO:

- setup A-MQ7 corresponding config
- setup A-MQ6 JDBC shared storage 
- setup A-MQ7 JDBC share storage

