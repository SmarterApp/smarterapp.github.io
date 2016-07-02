---
title: Program Management (ProgMan) Installation Checklist
permalink: "deployment/checklist/progman.html"
layout: "document"
categories: ["deployment", "checklist", "shared_services"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide configuration settings for TDS components |
| Communicates With | OpenAM<br>Permissions<br>ART<br>Proctor<br>Teacher Hand-Scoring System<br>TestSpecBank |
| Repository Location | [https://github.com/SmarterApp/SS_ProgramManagement](https://github.com/SmarterApp/SS_ProgramManagement){:target="_blank"} |
| Additional Documentation | [Program Management User Guide](http://www.smarterapp.org/documents/ProgramManagementUserGuide.pdf){:target="_blank"}<br>[ProgMan Technical Design](https://github.com/SmarterApp/SS_ProgramManagement/blob/master/docs/SBAC-11%20Prog%20man%20Technical%20Design.doc){:target="_blank"}<br>[API Documentation](https://github.com/SmarterApp/SS_ProgramManagement/tree/master/docs/api_docs){:target="_blank"}<br>[Design Diagrams](https://github.com/SmarterApp/SS_ProgramManagement/tree/master/docs/designPics){:target="_blank"}<br>[Sequence Diagrams](https://github.com/SmarterApp/SS_ProgramManagement/tree/master/docs/sequenceDiagrams){:target="_blank"} |

# Instructions

{% include checklist/mongodb_setup.md %}

## Create AWS Web Application Instance
* Create server instance to host the Program Management (ProgMan) component
  * Select an image with the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system
* Create or choose an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 22
  * 80
  * 443
  * 1043
  * 8080
  * 8084
  * 8443

## ProgMan Setup
* Update package manager:
  * `sudo apt-get update`
  * `sudo apt-get upgrade -y`

* Install packages to satisfy dependencies:
  * `sudo apt-get install -y ntp mercurial openjdk-7-jdk`

{% include checklist/tomcat_setup.md %}

{% include checklist/generate_keystore.md %}

### Deploy ProgMan Components

#### Configure Tomcat
* Stop the Tomcat service:
  * `sudo service tomcat7 stop`

{% include checklist/tomcat_java_opts.md %}

* Example:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>JAVA_OPTS="-Djava.awt.headless=true\
 -XX:+UseConcMarkSweepGC\
 -Xms<span class="placeholder-example">512m</span>\
 -Xmx<span class="placeholder-example">2048m</span>\
 -XX:PermSize=<span class="placeholder-example">512m</span>\
 -XX:MaxPermSize=<span class="placeholder-example">1512m</span>\
 -DSB11_CONFIG_DIR=$CATALINA_BASE/resources\
 -Dspring.profiles.active=mna.client.null,server.singleinstance,progman.client.impl.null,special.role.required"
 -Dprogman.baseUri=http://<span class="placeholder-example">54.213.81.243</span>/rest/\
 -Dprogman.locator=<span class="placeholder-example">pm</span>,<span class="placeholder-example">Development</span>"</code>
 </pre>
 </div>

* Create a directory for the ProgMan log files:
  * `sudo mkdir -p /usr/share/tomcat7/logs/{prog-mgmnt.webapp,prog-mgmnt.rest}`
  * `sudo chown -R tomcat7:tomcat7 /usr/share/tomcat7/logs/`
* ***OPTIONAL:***  Create links in the Tomcat log directory to the REST and Web Application log files:
  * `sudo ln -s /usr/share/tomcat7/logs/prog-mgmnt.webapp/prog-mgmnt.webapp.log /var/lib/tomcat7/logs/webapp.log`
  * `sudo ln -s /usr/share/tomcat7/logs/prog-mgmnt.rest/prog-mgmnt.rest.log /var/lib/tomcat7/logs/rest.log`

#### Download REST Component War File
* Download the latest `.war` file for the ProgMan REST Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://github.com/SmarterApp/SS_ProgramManagement/releases/download/R01.00.38/prog-mgmnt.rest-R01.00.38.war -O /var/lib/tomcat7/webapps/rest.war`

* Create a `rest-endpoints.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/rest-endpoints.properties`:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://[<span class="placeholder">FQDN or IP Address of ProgMan REST component, defeault port is 8080</span>]/rest
pm.rest.context.root=/rest/
pm.minJs=false</code>
</pre>
</div>
An example of a configured `rest-endpoints.properties`:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://<span class="placeholder-example">52.32.255.241:8080</span>/rest
pm.rest.context.root=/rest/
pm.minJs=false</code>
</pre>
</div>

#### Download ProgMan Web Application Component
* Download the latest `.war` file for the ProgMan Web Application Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://github.com/SmarterApp/SS_ProgramManagement/releases/download/R01.00.38/prog-mgmnt.webapp-R01.00.38.war -O /var/lib/tomcat7/webapps/ROOT.war`
* Create a `progman-bootstrap.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/progman-bootstrap.properties`:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>#mna.properties
progman.mna.description="The Program Management Component ([<span class="placeholder">environment name</span>])"
#mna.mnaUrl=https://your.mna.server/rest
#mna.logger.level=INFO
#mna.oauth.batch.account=mna-client-email-address
#mna.oauth.batch.password=mna-client-password
#mongo.properties
#placeholder for mongo settings - note: do not check in real credentials
pm.mongo.hostname=[<span class="placeholder">FQDN or IP address of MongoDB server</span>]
pm.mongo.port=[<span class="placeholder">port that MongoDB listens on, default is 27017</span>]
pm.mongo.user=[<span class="placeholder">mongo user name, mongo_admin if following this checklist</span>]
pm.mongo.password=[<span class="placeholder">password for mongo_admin user account</span>]
pm.mongo.dbname=[<span class="placeholder">Name of database created for ProgMan in MongoDB</span>]
#pbe.properties
pm.pbe.pass=password123
#pm.pbe.pass=secret-salt
#rest-endpoints.properties
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://[<span class="placeholder">FQDN or IP address of AWS instance hosting ProgMan REST component, default port is 8080</span>]/rest
pm.minJs=false
pm.rest.context.root=/rest/
###########################
# pm-security.properties
###########################
#security props
pm.security.saml.keystore.user=[<span class="placeholder">alias of private key stored in samlKeystore.jks</span>]
pm.security.saml.keystore.pass=[<span class="placeholder">password for samlKeystore.jks</span>]
pm.security.dir=file:///[<span class="placeholder">path to samlKeystore.jks, use <strong>/var/lib/tomcat7/resources/security</strong> if following this checklist</span>]
pm.rest.saml.metadata.filename=[<span class="placeholder">name of SAML metadata file for REST component</span>]
pm.webapp.saml.metadata.filename=[<span class="placeholder">name of SAML metadata file for web application component</span>]
component.name=ProgramManagement
pm.oauth.checktoken.endpoint=https://[<span class="placeholder">load balancer for OpenAM</span>]/auth/oauth2/tokeninfo?realm=/sbac
pm.security.idp=https://[<span class="placeholder">load balancer for OpenAM</span>]/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac
permission.uri=http://[<span class="placeholder">FQDN or IP address of Permissions application. <strong>NOTE: </strong> the Permissions program has not been installed yet.  This can be configured after Permissions has been deployed; ProgMan should still start up</span>]/rest

logfile.path=/var/log/tomcat7/</code>
</pre>
</div>

An example of a configured `progman-bootstrap.properties`:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>#mna.properties
progman.mna.description="The Program Management Component (<span class="placeholder-example">Development</span>)"
#mna.mnaUrl=https://your.mna.server/rest
#mna.logger.level=INFO
#mna.oauth.batch.account=mna-client-email-address
#mna.oauth.batch.password=mna-client-password
#mongo.properties
#placeholder for mongo settings - note: do not check in real credentials
pm.mongo.hostname=<span class="placeholder-example">52.32.123.173</span>
pm.mongo.port=<span class="placeholder-example">27017</span>
pm.mongo.user=<span class="placeholder-example">mongo_admin</span>
pm.mongo.password=<span class="placeholder-example">[redacted]</span>
pm.mongo.dbname=<span class="placeholder-example">progman</span>
#pbe.properties
pm.pbe.pass=<span class="placeholder-example">[redacted]</span>
#pm.pbe.pass=secret-salt
#rest-endpoints.properties
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=<span class="placeholder-example">http://52.34.140.123:8080/rest</span>
pm.minJs=false
pm.rest.context.root=/rest/
###########################
# pm-security.properties
###########################
#security props
pm.security.saml.keystore.user=<span class="placeholder-example">progman-saml-sp</span>
pm.security.saml.keystore.pass=<span class="placeholder-example">[redacted]</span>
pm.security.dir=file:////<span class="placeholder-example">var/lib/tomcat7/resources/security</span>
pm.rest.saml.metadata.filename=<span class="placeholder-example">rest_metadata.xml</span>
pm.webapp.saml.metadata.filename=<span class="placeholder-example">web_metadata.xml</span>
component.name=ProgramManagement
pm.oauth.checktoken.endpoint=<span class="placeholder-example">https://sso-dev.sbtds.org/auth/oauth2/tokeninfo?realm=/sbac</span>
pm.security.idp=<span class="placeholder-example">https://sso-dev.sbtds.org/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac</span>
permission.uri=<span class="placeholder-example">http://52.32.19.35:8080/rest</span>

logfile.path=/var/log/tomcat7/</code>
</pre>
</div>

***IMPORTANT:***{: style="color: #f00;"} <span style="background-color: #ff0;">Conduct the **SAML Setup and Configuration** for the REST component *and* Web Application Component.  After completing the SAML Setup and Configuration steps, there should be two metadata files</span>:

* A SAML XML metadata file for the REST component, located where-ever the file name/path is configured for `pm.security.dir` and `pm.rest.saml.metadata.filename` (e.g. <span class="placeholder-example">/var/lib/tomcat7/resources/security/rest_metadata.xml</span>)
* A SAML XML metadata file for the web application component located where-ever the file name/path is configured for `pm.security.dir` and `pm.webapp.saml.metadata.filename` (e.g. <span class="placeholder-example">/var/lib/tomcat7/resources/security/web_metadata.xml</span>)

{% include checklist/saml_setup.md %}

### Update ProgMan Bootstrap Properties
* Update the following lines of the `progman-bootstrap.properties` to use the correct SAML metadata files:
  * `pm.rest.saml.metadata.filename=`[*name of the SAML metadata file for the REST component*{: style="color: #f00;"}]
  * `pm.webappt.saml.metadata.filename=`[*name of the SAML metadata file for the web application component*{: style="color: #f00;"}]

{% include checklist/saml_registration.md %}

### Load Seed Data into ProgMan
***IMPORTANT:*** MongoDB must be installed on whatever computer runs the script to load the ProgMan seed data.

* Unless already done, clone the `TDS_Build` repository from GitHub:
  * `git clone https://github.com/SmarterApp/TDS_Build.git`
* Navigate to the directory where the seed data script is located:
  * `cd `[*Path to where the `TDS_Build` repository was cloned*{: style="color: #f00;"}]`/database/mongodb/progman`
  * Example:
    * `cd `<span class="placeholder-example">~/dev/ucla/sbac/sbrepo/TDS_Build/database/mongodb/progman</span>
* Edit the `load-seed-data.sh` script to configure the following:
  * `HOST=`[*The FQDN or IP address of the MongoDB server hosting the ProgMan database*{: style="color: #f00;"}]
  * `PORT=`[*The port on which MongoDB is listening*{: style="color: #f00;"}]
  * `USER=`[*The user account with "readWrite" privileges in the ProgMan database*{: style="color: #f00;"}]
  * `PW=`[*The password for the user account*{: style="color: #f00;"}]
  * `DB=`[*The name of the database containing ProgMan's data*{: style="color: #f00;"}]
* Example:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>HOST=<span class="placeholder-example">54.201.173.209</span>     # The FQDN or IP address of the MongoDB server hosting the ProgMan database
PORT=<span class="placeholder-example">27017</span>              # The port on which MongoDB is listening
USER=<span class="placeholder-example">admin</span>              # The user account with "readWrite" privileges in the ProgMan database
PW=<span class="placeholder-example">[redacted]</span>          # The password for the user account
DB=<span class="placeholder-example">progman</span>              # The name of the database containing ProgMan's data</code>
</pre>
</div>

* Execute the `load-seed-data.sh` script:
  * `./load-seed-data.sh`

## Verification
* Log into ProgMan with the Prime User account created during the OpenDJ Verification process
  * **NOTE:** If this is the first time using the Prime User account, you may be prompted to change the password and set up the security questions
* Verify the home page of ProgMan appears
* Click on the **Manage Components** link in navigation menu on the left rail
* Verify records are displayed

[back to Deployment Checklists](index.html)