---
title: Program Management (ProgMan) Installation Checklist
permalink: "deployment/checklist/progman.html"
layout: "document"
categories: ["deployment", "checklist"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide configuration settings for TDS components |
| Communicates With | OpenAM, Permissions, ART, Proctor, Teacher Hand-Scoring System, TestSpecBank |
| Repository Location | [https://bitbucket.org/sbacoss/programmanagement_release](https://bitbucket.org/sbacoss/programmanagement_release) |
| Additional Documentation | [Program Management User Guide](Program_Management_User_Guide.docx), [ProgMan Technical Design](SBAC-11 Prog man Technical Design.doc), [API Documentation](https://bitbucket.org/sbacoss/programmanagement_release/src/5aebb91fd5bb08fc841e7427b7ee789d36fb1cb0/docs/api_docs/?at=default), [Design Diagrams](https://bitbucket.org/sbacoss/programmanagement_release/src/5aebb91fd5bb08fc841e7427b7ee789d36fb1cb0/docs/designPics/?at=default), [Sequence Diagrams](https://bitbucket.org/sbacoss/programmanagement_release/src/5aebb91fd5bb08fc841e7427b7ee789d36fb1cb0/docs/sequenceDiagrams/?at=default) |

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

~~~~
JAVA_OPTS="-Djava.awt.headless=true\
 -XX:+UseConcMarkSweepGC\
 -Xms512m\
 -Xmx2048m\
 -XX:PermSize=512m\
 -XX:MaxPermSize=1512m\
 -DSB11_CONFIG_DIR=$CATALINA_BASE/resources\
 -Dspring.profiles.active=mna.client.null,server.singleinstance,progman.client.impl.null,special.role.required"
~~~~

* Create a directory for the ProgMan log files:
  * `sudo mkdir -p /usr/share/tomcat7/logs/{prog-mgmnt.webapp,prog-mgmnt.rest}`
  * `sudo chown -R tomcat7:tomcat7 /usr/share/tomcat7/logs/prog-mgmnt.webapp/`
* ***OPTIONAL:***  Create links in the Tomcat log directory to the REST and Web Application log files:
  * `sudo ln -s /usr/share/tomcat7/logs/prog-mgmnt.webapp/prog-mgmnt.webapp.log /var/lib/tomcat7/logs/webapp.log`
  * `sudo ln -s /usr/share/tomcat7/logs/prog-mgmnt.rest/prog-mgmnt.rest.log /var/lib/tomcat7/logs/rest.log`

#### Download REST Component War File
* Download the latest `.war` file for the ProgMan REST Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://bitbucket.org/fwsbac/programmanagement_release/downloads/prog-mgmnt.rest-R01.00.38.war -O /var/lib/tomcat7/webapps/rest.war`
* Create a `rest-endpoints.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/rest-endpoints.properties`:

***TODO:  Is this file even in use?***

<div class="highlighter-rouge">
<pre class="highlight">
<code>#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://[<span class="placeholder">FQDN or IP Address of ProgMan REST component, defeault port is 8080</span>]/rest
pm.rest.context.root=/rest/
pm.minJs=false</code>
</pre>
</div>
An example of a configured `rest-endpoints.properties`:

~~~~
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://52.32.255.241:8080/rest
pm.rest.context.root=/rest/
pm.minJs=false
~~~~

#### Download ProgMan Web Application Component
* Download the latest `.war` file for the ProgMan Web Application Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://bitbucket.org/fwsbac/programmanagement_release/downloads/prog-mgmnt.webapp-R01.00.38.war -O /var/lib/tomcat7/webapps/ROOT.war`
* Create a `progman-bootstrap.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/progman-bootstrap.properties`:

<pre class="highlight">
<code>
#mna.properties
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
pm.mongo.dbname=progman
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

logfile.path=/var/log/tomcat7/
</code>
</pre>

An example of a configured `progman-bootstrap.properties`:

~~~~
#mna.properties
progman.mna.description="The Program Management Component (Development)"
#mna.mnaUrl=https://your.mna.server/rest
#mna.logger.level=INFO
#mna.oauth.batch.account=mna-client-email-address
#mna.oauth.batch.password=mna-client-password
#mongo.properties
#placeholder for mongo settings - note: do not check in real credentials
pm.mongo.hostname=52.32.123.173
pm.mongo.port=27017
pm.mongo.user=mongo_admin
pm.mongo.password=[redacted]
pm.mongo.dbname=progman
#pbe.properties
pm.pbe.pass=[redacted]
#pm.pbe.pass=secret-salt
#rest-endpoints.properties
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://52.34.140.123:8080/rest
pm.minJs=false
pm.rest.context.root=/rest/
###########################
# pm-security.properties
###########################
#security props
pm.security.saml.keystore.user=progman-saml-sp
pm.security.saml.keystore.pass=[redacted]
pm.security.dir=file:////var/lib/tomcat7/resources/security
pm.rest.saml.metadata.filename=rest_metadata.xml
pm.webapp.saml.metadata.filename=web_metadata.xml
component.name=ProgramManagement
pm.oauth.checktoken.endpoint=https://sso-dev.sbtds.org/auth/oauth2/tokeninfo?realm=/sbac
pm.security.idp=https://sso-dev.sbtds.org/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac
permission.uri=http://52.32.19.35:8080/rest

logfile.path=/var/log/tomcat7/
~~~~

***IMPORTANT:***{: style="color: #f00;"} <span style="background-color: #ff0;">Conduct the **SAML Setup and Configuration** for the REST component *and* Web Application Component.  After completing the SAML Setup and Configuration steps, there should be two metadata files</span>:

* A SAML XML metadata file for the REST component, located where-ever the file name/path is configured for `pm.security.dir` and `pm.rest.saml.metadata.filename` (e.g. `/var/lib/tomcat7/resources/security/rest_metadata.xml`)
* A SAML XML metadata file for the web application component located where-ever the file name/path is configured for `pm.security.dir` and `pm.webapp.saml.metadata.filename` (e.g. `/var/lib/tomcat7/resources/security/web_metadata.xml`)

{% include checklist/saml_setup.md %}

### Update ProgMan Bootstrap Properties
* Update the following lines of the `progman-bootstrap.properties` to use the correct SAML metadata files:
  * `pm.rest.saml.metadata.filename=`[*name of the SAML metadata file for the REST component*{: style="color: #f00;"}]
  * `pm.webappt.saml.metadata.filename=`[*name of the SAML metadata file for the web application component*{: style="color: #f00;"}]

{% include checklist/saml_registration.md %}

### Load Seed Data into ProgMan
***IMPORTANT:*** MongoDB must be installed on whatever computer runs the script to load the ProgMan seed data.

* Unless already done, clone the `tds-build` repository from BitBucket:
  * `git clone https://`[*Your BitBucket account or team name*{: style="color: #f00;"}]`/fwsbac/tds-build.git`
  * Example:
    * `git clone https://jjohnson-fwtech@bitbucket.org/fwsbac/tds-build.git`
* Navigate to the directory where the seed data script is located:
  * `cd `[*Path to where the `tds-build` repository was cloned*{: style="color: #f00;"}]`/database/mongodb/progman`
  * Example:
    * `cd ~/dev/ucla/sbac/sbrepo/tds-build/database/mongodb/progman`
* Edit the `load-seed-data.sh` script to configure the following:
  * `HOST=`[*The FQDN or IP address of the MongoDB server hosting the ProgMan database*{: style="color: #f00;"}]
  * `PORT=`[*The port on which MongoDB is listening*{: style="color: #f00;"}]
  * `USER=`[*The user account with "readWrite" privileges in the ProgMan database*{: style="color: #f00;"}]
  * `PW=`[*The password for the user account*{: style="color: #f00;"}]
  * `DB=`[*The name of the database containing ProgMan's data*{: style="color: #f00;"}]
* Example:

~~~~
HOST=54.201.173.209     # The FQDN or IP address of the MongoDB server hosting the ProgMan database
PORT=27017              # The port on which MongoDB is listening
USER=admin              # The user account with "readWrite" privileges in the ProgMan database
PW=[redacted]          # The password for the user account
DB=progman              # The name of the database containing ProgMan's data
~~~~

* Execute the `load-seed-data.sh` script:
  * `./load-seed-data.sh`

## Verification
* Log into ProgMan with the Prime User account created during the OpenDJ Verification process
  * **NOTE:** If this is the first time using the Prime User account, you may be prompted to change the password and set up the security questions
* Verify the home page of ProgMan appears
* Click on the **Manage Components** link in navigation menu on the left rail
* Verify records are displayed

[back to Deployment Checklists](index.html)