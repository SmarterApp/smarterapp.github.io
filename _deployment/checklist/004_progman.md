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

#### Build ProgMan Components
* On a suitable build machine, get the latest source code for ProgMan
* Build the programmanagement_release repository (which builds the REST and Web Application components):
  * `mvn -q clean install -DskipTests -DXmx2048m -DXX:MaxPermSize=1024m -f [path/to/pom.xml]`
  * Example:
    * `mvn -q clean install -DskipTests -DXmx2048m -DXX:MaxPermSize=1024m -f /Users/jjohnson/dev/ucla/sbac/sbrepo/repositories/programmanagement_release/pom.xml`
  * After Maven finishes the build, the desired `.war` files will be in their respective `target` directories

#### Deploy ProgMan REST Component
* From the build machine, copy the ProgMan REST component (the `.war` file) to the AWS instance:
  * `scp -i [path/to/key] [path/to/progman-rest.war] ubuntu@[fqdn or ip address of AWS isntance]:~/rest.war`
  * Example:
    * `scp -i ~/.ssh/tds/ssh-dev.pem prog-mgmnt.rest-0.0.3-SNAPSHOT.war ubuntu@54.213.81.243:~/rest.war`
* On the AWS instance, move the `rest.war` into the Tomcat server's `webapps` directory:
  * `sudo mv ~/rest.war /var/lib/tomcat7/webapps`
* Create a `rest-endpoints.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/rest-endpoints.properties`:

***TODO:  Is this file even in use?***

~~~~
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://[FQDN or IP Address of ProgMan REST component, defeault port is 8080]/rest
pm.rest.context.root=/rest/
pm.minJs=false
~~~~

An example of a configured `rest-endpoints.properties`:

~~~~
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://52.32.255.241:8080/rest
pm.rest.context.root=/rest/
pm.minJs=false
~~~~

#### Deploy ProgMan Web Application Component
* From the build machine, copy the ProgMan REST component (the `.war` file) to the AWS instance:
  * `scp -i [path/to/key] [path/to/progman-rest.war] ubuntu@[fqdn or ip address of AWS isntance]:~/ROOT.war`
  * Example:
    * `scp -i ~/.ssh/tds/ssh-dev.pem prog-mgmnt.webapp-0.0.3-SNAPSHOT.war ubuntu@54.213.81.243:~/ROOT.war`
* On the AWS instance, move the `ROOT.war` into the Tomcat server's `webapps` directory:
  * `sudo mv ~/ROOT.war /var/lib/tomcat7/webapps`
* Create a `progman-bootstrap.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/progman-bootstrap.properties`:

~~~~
#mna.properties
progman.mna.description="The Program Management Component (environment name)"
#mna.mnaUrl=https://your.mna.server/rest
#mna.logger.level=INFO
#mna.oauth.batch.account=mna-client-email-address
#mna.oauth.batch.password=mna-client-password
#mongo.properties
#placeholder for mongo settings - note: do not check in real credentials
pm.mongo.hostname=[FQDN or IP address of MongoDB server]
pm.mongo.port=[port that MongoDB listens on, default is 27017]
pm.mongo.user=[mongo user name, mongo_admin if following this checklist]
pm.mongo.password=[password for mongo_admin user account]
pm.mongo.dbname=progman
#pbe.properties
pm.pbe.pass=password123
#pm.pbe.pass=secret-salt
#rest-endpoints.properties
#base URLs for REST endpoints, replace with URLs that work for the server this is being run on
pm.rest.service.endpoint=http://[FQDN or IP address of AWS instance hosting ProgMan REST component, default port is 8080]/rest
pm.minJs=false
pm.rest.context.root=/rest/
###########################
# pm-security.properties
###########################
#security props
pm.security.saml.keystore.user=[alias of private key stored in samlKeystore.jks]
pm.security.saml.keystore.pass=[password for samlKeystore.jks]
pm.security.dir=file:///[path to samlKeystore.jks, use /var/lib/tomcat7/resources/security if following this checklist]
pm.rest.saml.metadata.filename=[name of SAML metadata file for REST component]
pm.webapp.saml.metadata.filename=[name of SAML metadata file for web application component]
component.name=ProgramManagement
pm.oauth.checktoken.endpoint=https://[load balancer for OpenAM]/auth/oauth2/tokeninfo?realm=/sbac
pm.security.idp=https://[load balancer for OpenAM]/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac
permission.uri=http://[FQDN or IP address of Permissions application]/rest

logfile.path=/var/log/tomcat7/
~~~~

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

**NOTE:** Conduct the SAML Setup and Configuration for the REST component and Web Application Component.  After completing the SAML Setup and Configuration steps, there should be two metadata files:

* A SAML XML metadata file for the REST component (e.g. `/var/lib/tomcat7/resources/security/rest_metadata.xml`)
* A SAML XML metadata file for the web application component (e.g. `/var/lib/tomcat7/resources/security/web_metadata.xml`)

{% include checklist/saml_setup.md %}

### Update ProgMan Bootstrap Properties
* Update the following lines of the `progman-bootstrap.properties` to use the correct SAML metadata files:
  * pm.rest.saml.metadata.filename=[name of the SAML metadata file for the REST component]
  * pm.webappt.saml.metadata.filename=[name of the SAML metadata file for the web application component]

{% include checklist/saml_registration.md %}

## Verification
* Log into ProgMan with the Prime User account created during the OpenDJ Verification process
  * **NOTE:** If this is the first time using the Prime User account, you may be prompted to change the password and set up the security questions
* Verify the home page of ProgMan appears

[back to Deployment Checklists](index.html)