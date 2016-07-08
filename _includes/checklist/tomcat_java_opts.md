* Edit the `/etc/default/tomcat7` file, updating the `JAVA_OPTS` value to what's shown below:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>JAVA_OPTS="-Djava.awt.headless=true\
 -XX:+UseConcMarkSweepGC\
 -Xms[<span class="placeholder">initial amount of memory that can be allocated to the JVM heap</span>]\
 -Xmx[<span class="placeholder">maximum amount of memory that can be allocated to the JVM heap</span>]\
 -XX:PermSize=[<span class="placeholder">initial amount of memory that can be used for PermGen</span>]\
 -XX:MaxPermSize=[<span class="placeholder">maximum amount of memory that can be used for PermGen</span>]\
 -DSB11_CONFIG_DIR=$CATALINA_BASE/resources\
 -Dspring.profiles.active=progman.client.impl.integration,mna.client.null,server.singleinstance\
 -Dprogman.baseUri=http://[<span class="placeholder">URL to the ProgMan REST component</span>]/rest/\
 -Dprogman.locator=[<span class="placeholder">name of component in ProgMan</span>],[<span class="placeholder">name of Component's environment in ProgMan</span>]"</code>
</pre>
</div>

* **NOTE:** If the component being set up will be load-balanced, then change the `server.singleinstance` (for the `spring.profiles.active` option) to `server.loadbalanced`.