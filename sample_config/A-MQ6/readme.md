
Configuration example templates that are based on customer config. The configs are defined for various activemq distros (standalone, karaf bases) but could be used as a template for A-MQ 6 configuration. They have been santised of customer identifying data.

`activemq_001.xml` Demonstrates using a JDBC persistence adapter

`activemq_002.xml` Demonstrates many of the common usages:
 		- network connector
        - some transport connector attributes
        - SSL
        - Various broker plugins
        - Scheduler 
        - mKahadb
        - Security

`activemq_003.xml` Demonstrates using virtual destinations

`activemq_004.xml` Demonstrates using:
   - additional broker attributes 
   - additional destination policy attributes
   - temp storage configuration
