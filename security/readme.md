## Security

### Basic

#### A-MQ 6

Example in `security/basic/a-mq-6/`

Define users in `etc/users.properties`

	admin=admin,admins,users,guests
	user=user,users
	guest=guest,guests

Define authorization in `etc/activemq.xml`

            <authorizationPlugin>
              <map>
                <authorizationMap groupClass="org.apache.karaf.jaas.boot.principal.RolePrincipal">
                  <authorizationEntries>
                    <authorizationEntry queue=">" read="admins" write="admins" admin="admins"/>
                    <authorizationEntry queue="USERS.>" read="users" write="users" admin="users"/>
                    <authorizationEntry queue="GUEST.>" read="guests" write="guests,users" admin="guests,users"/>
                    <authorizationEntry topic=">" read="admins" write="admins" admin="admins"/>
                    <authorizationEntry topic="USERS.>" read="users" write="users" admin="users"/>
                    <authorizationEntry topic="GUEST.>" read="guests" write="guests,users" admin="guests,users"/>
                    <authorizationEntry topic="ActiveMQ.Advisory.>" read="guests,users" write="guests,users" admin="guests,users"/>
                  </authorizationEntries>
                  <!-- let's assign roles to temporary destinations. comment this entry if we don't want any roles assigned to temp destinations  -->
                  <tempDestinationAuthorizationEntry>
                    <tempDestinationAuthorizationEntry read="tempDestinationAdmins" write="tempDestinationAdmins" admin="tempDestinationAdmins"/>
                  </tempDestinationAuthorizationEntry>
                </authorizationMap>
              </map>
            </authorizationPlugin>


Test

Start broker

	cd security/basic/a-mq-6
	bin/start

Run different scenarios

	java -jar /opt/a-mq-6/activemq-all.jar consumer --user admin --password admin --destination TEST

OK
	
	java -jar /opt/a-mq-6/activemq-all.jar consumer --user user --password user --destination TEST

Forbidden
	
	java -jar /opt/a-mq-6/activemq-all.jar consumer --user user --password user --destination USERS.TEST

OK


### A-MQ 7

Example in `security/basic/a-mq-7/`


Define users in `etc/artemis-users.properties`

	admin=admin
	user=user
	guest=guest

Define roles in `etc/artemis-roles.properties`

	admins=admin
	users=user,admin
	guests=guest,admin

Configure authorization policies in `etc/broker.xml`	

      <security-settings>
         <security-setting match="jms.queue.#">
            <permission type="createNonDurableQueue" roles="admins"/>
            <permission type="deleteNonDurableQueue" roles="admins"/>
            <permission type="createDurableQueue" roles="admins"/>
            <permission type="deleteDurableQueue" roles="admins"/>
            <permission type="consume" roles="admins"/>
            <permission type="browse" roles="admins"/>
            <permission type="send" roles="admins"/>
            <!-- we need this otherwise ./artemis data imp wouldn't work -->
            <permission type="manage" roles="admins"/>
         </security-setting>

         <security-setting match="jms.queue.USERS.#">
            <permission type="createNonDurableQueue" roles="users"/>
            <permission type="deleteNonDurableQueue" roles="users"/>
            <permission type="createDurableQueue" roles="users"/>
            <permission type="deleteDurableQueue" roles="users"/>
            <permission type="consume" roles="users"/>
            <permission type="browse" roles="users"/>
            <permission type="send" roles="users"/>
            <!-- we need this otherwise ./artemis data imp wouldn't work -->
            <permission type="manage" roles="users"/>
         </security-setting>         

         <security-setting match="jms.queue.GUESTS.#">
            <permission type="createNonDurableQueue" roles="guests"/>
            <permission type="deleteNonDurableQueue" roles="guests"/>
            <permission type="createDurableQueue" roles="guests"/>
            <permission type="deleteDurableQueue" roles="guests"/>
            <permission type="consume" roles="guests"/>
            <permission type="browse" roles="guests"/>
            <permission type="send" roles="guests"/>
            <!-- we need this otherwise ./artemis data imp wouldn't work -->
            <permission type="manage" roles="guests"/>
         </security-setting>                  
      </security-settings>	

Test

Start broker

	cd security/basic/a-mq-7
	bin/artemis start

Run different scenarios

	java -jar /opt/a-mq-6/activemq-all.jar consumer activemq:consumer --user admin --password admin --destination TEST

OK

	java -jar /opt/a-mq-6/activemq-all.jar consumer activemq:consumer --user user --password user --destination TEST

Forbidden

	java -jar /opt/a-mq-6/activemq-all.jar consumer activemq:consumer --user user --password user --destination USERS.TEST

OK     