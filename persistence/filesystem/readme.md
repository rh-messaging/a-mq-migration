
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

##Questions:

- does A-MQ7 offer something similar to mkahadb
- what file systems (particularly network shared filesystems) is A-MQ7 supported on?

##TODO
- create an corresponding A-MQ7 config


