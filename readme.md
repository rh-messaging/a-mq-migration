# A-MQ Migration Guide

## Prerequisite

A-MQ 6 installed at `/opt/a-mq-6`

A-MQ 6 installed at `/opt/a-mq-6_dist2/` - required for examples that require 2 A-MQ installation running on the one machine. The Karaf A-MQ install only allow one root container instance to be running at one time. The simplest approach is to install 2 A-MQ distributions. 

A-MQ 7 installed at `/opt/a-mq-7`

Openwire client for testing

	unzip /opt/a-mq-6/extras/apache-activemq-${VERSION}.zip
	ln -s /opt/a-mq-6/extras/apache-activemq-${VERSION}/activemq-all-${VERSION}.jar /opt/a-mq-6/activemq-all.jar

Suite installed in `/workspace/oss/a-mq-migration/`

Topics:

* [Securing Broker](security)
* [Creating Network Bridge](network)



