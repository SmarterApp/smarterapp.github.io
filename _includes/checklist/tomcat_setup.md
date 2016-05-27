### Set Up Tomcat Server
* Remove `apparmor`:
  * `sudo /etc/init.d/apparmor stop`
  * `sudo update-rc.d -f apparmor remove`
  * `sudo apt-get --purge remove -y apparmor apparmor-utils libapparmor-perl libapparmor1`
* Install Tomcat Server (if not installed already):
  * `sudo apt-get install -y tomcat7`
* Stop the Tomcat service:
  * `sudo service tomcat7 stop`
* Remove the `ROOT`  directory:
  * `sudo rm -rf /var/lib/tomcat7/webapps/ROOT`
* Update the `server.xml` to allow for large HTTP Headers:
  * Edit the `/etc/tomcat7/server.xml` file
  * Find the `<Connector>` element
  * Add the following attribute and value to the `<Connector>` element:
    * `maxHttpHeaderSize="65536"`
  * Example of an updated `<Connector>` element:
<div class="highlighter-rouge">
<pre class="highlight">
<code>
     &lt;Connector port="8080" protocol="HTTP/1.1"
          connectionTimeout="20000"
          URIEncoding="UTF-8"
          redirectPort="8443"
          <span class="placeholder-example">maxHttpHeaderSize="65536"</span> /&gt;
</code>
</pre>
</div>
