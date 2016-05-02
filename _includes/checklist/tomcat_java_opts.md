* Edit the `/etc/default/tomcat` file, updating the `JAVA_OPTS` value to what's shown below:

~~~~
JAVA_OPTS="-Djava.awt.headless=true\
 -XX:+UseConcMarkSweepGC\
 -Xms[initial amount of memory that can be allocated to the JVM heap]\
 -Xmx[maximum amount of memory that can be allocated to the JVM heap]\
 -XX:PermSize=[initial amount of memory that can be used for PermGen]\
 -XX:MaxPermSize=[maximum amount of memory that can be used for PermGen]\
 -DSB11_CONFIG_DIR=$CATALINA_BASE/resources\
 -Dspring.profiles.active=progman.client.impl.integration,mna.client.null,server.singleinstance\
 -Dprogman.baseUri=http://[URL to the ProgMan REST component]/rest/\
 -Dprogman.locator=[name of component in ProgMan],[name of Component's environment in ProgMan]"
~~~~

* **NOTE:** If the component being set up will be load-balanced, then change the `server.singleinstance` (for the `spring.profiles.active` option) to `server.loadbalanced`.