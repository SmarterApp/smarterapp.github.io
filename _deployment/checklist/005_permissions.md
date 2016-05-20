---
title: Permissions Installation Checklist
permalink: "deployment/checklist/permissions.html"
layout: "document"
categories: ["deployment", "checklist", "shared_services"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide component and role authorization settings for TDS components |
| Communicates With | ProgMan, ART, Proctor, Teacher Hand-Scoring System, TestSpecBank |
| Repository Location | [https://bitbucket.org/sbacoss/prermissions_release](https://bitbucket.org/sbacoss/permissions_release) |
| Additional Documentation | [Permissions DTD](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/DTD.txt?at=default), [Permissions API](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/Permissions-API.pdf?at=default), [Permissions Database Design](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/PermissionsDB_Design.docx?at=default), [Permissions User Guide](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/PermissionsUserGuide.pdf?at=default), [Permissions High-Level Design](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/SBAC11%20High%20Level%20Design-Permissions.docx?at=default), [Permissions Roles to Components Map](https://bitbucket.org/sbacoss/permissions_release/src/5efe2172013d5784161097decf9d1c73cc45fb73/Documents/permissions-roles-components.xlsx?at=default) |

# Instructions

{% include checklist/mysql_setup.md %}

### Create Permissions Database
* Install git and mercurial:
  * `sudo apt-get install -y git mercurial`
* Clone the permissions_release repository:
  * `hg clone https://`[*your bitbucket user name*{: style="color: #f00;"}]`@bitbucket.org/sbacoss/permissions_release`
    * Example:  `hg clone https://jjohnson-fwtech@bitbucket.org/sbacoss/permissions_release`
* Clone the Fairway `tds-build` repository:
  * `git clone https://`[*your bitbucket username*{: style="color: #f00;"}]`@bitbucket.org/fwsbac/tds-build.git`
    * Example:  `git clone https://jjohnson-fwtech@bitbucket.org/fwsbac/tds-build.git`
* ***NOTE:*** When cloning the repositories above, they should be "siblings" at the same level.  For example, if both
repositories are cloned in the `ubuntu` user's home directory, the directory will look like this:

~~~~
ubuntu@perm-db-deploy:~$ ls -alh
total 44K
drwxr-xr-x  6 ubuntu ubuntu 4.0K Apr 11 04:25 .
drwxr-xr-x  3 root   root   4.0K Apr 10 18:08 ..
-rw-r--r--  1 ubuntu ubuntu  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 ubuntu ubuntu 3.6K Apr 11 03:22 .bashrc
drwx------  2 ubuntu ubuntu 4.0K Apr 11 03:22 .cache
-rw-------  1 ubuntu ubuntu  193 Apr 11 04:20 .mysql_history
drwxrwxr-x  5 ubuntu ubuntu 4.0K Apr 11 04:25 permissions_release
-rw-r--r--  1 ubuntu ubuntu  675 Apr  9  2014 .profile
drwx------  2 ubuntu ubuntu 4.0K Apr 11 04:09 .ssh
drwxrwxr-x 13 ubuntu ubuntu 4.0K Apr 11 04:16 tds-build
-rw-------  1 ubuntu ubuntu 4.0K Apr 11 04:16 .viminfo
~~~~

* Navigate to the `tds-build` directory:
  * `cd tds-build`
* Update the `db-perm-schema-setup` script to use the correct user name and password (lines 20 and 21):
  * `USER=`[*A valid MySQL user name that can create databases*{: style="color: #f00;"}]
  * `PW=`[*The MySQL user's password*{: style="color: #f00;"}]
* If necessary, update the port and hostame (lines 18 and 19)
* Make the `db-perm-schema-setup.sh` executable:
  * `sudo chmod u+x db-perm-schema-setup.sh`
* Run the `db-perm-schema-setup.sh` script to create the Permissions database schema and load it with seed data:
  * `./db-perm-schema-setup.sh`

#### Verify the Permissions Database Schema Was Created and Populated With Seed Data
* Log into MySQL:
  * `mysql -u root -p`
* Execute the following query:
  * `show databases;`
* Output should appear as follows:

~~~~
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| permissions_db     |
+--------------------+
~~~~

* Change to the `permissions_db` database:
  * `use permissions_db;`
* List the tables in the database:
  * `show tables;`

~~~~
+--------------------------+
| Tables_in_permissions_db |
+--------------------------+
| component                |
| entitytype               |
| permission               |
| permission_role          |
| role                     |
| role_entity              |
+--------------------------+
~~~~

* Get number of records in the `permission_role` table:
  * `select count(*) from permission_role;`

~~~~
+----------+
| count(*) |
+----------+
|      416 |
+----------+
~~~~

* ***OPTIONAL:*** Get record counts from other tables in the `permissions_db`.  All of the tables should have records in them

## Create AWS Web Application Instance
* Create server instance to host the Permissions component
  * Select an image with the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system
* Create or choose an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 22
  * 80
  * 443
  * 1043
  * 8080
  * 8084
  * 8443

## Permissions Setup
* Update package manager:
  * `sudo apt-get update`
  * `sudo apt-get upgrade -y`

* Install packages to satisfy dependencies:
  * `sudo apt-get install -y ntp mercurial openjdk-7-jdk`

{% include checklist/tomcat_setup.md %}

{% include checklist/generate_keystore.md %}

### Configure Permissions in ProgMan
{% include checklist/progman_config.md %}

* Shown below are the Permissions properties that need to be configured in ProgMan:

* `datasource.url=`jdbc:mysql://[*FQDN or IP address of the MySQL server that hosts the Permissions database*{: style="color: red"}]/permissions_db
* `datasource.username=`[*MySQL user account that has access to read/write in the Permissions database*{: style="color: red"}]
* `datasource.password=`[*Password for the MySQL user account*{: style="color: red"}]
* `datasource.driverClassName=`com.mysql.jdbc.Driver
* `datasource.minPoolSize=`5
* `datasource.acquireIncrement=`5
* `datasource.maxPoolSize=`20
* `datasource.checkoutTimeout=`60000
* `datasource.maxConnectionAge=`0
* `datasource.acquireRetryAttempts=`5
* `permission.uri=`http://[*FQDN or IP address for the Permissions application*{: style="color: red"}]/rest
* `permission.security.profile=`[*The name of the environment, starting value is "Development"*{: style="color: red"}]
* `component.name=`Permissions
* `permission.security.idp=`https://[*FQDN or IP address of the OpenAM server*{: style="color: red"}]/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac
* `permission.webapp.saml.metadata.filename=`[*The name of the file that will contain the SAML for the Permissions component*{: style="color: red"}]
* `permission.security.saml.keystore.pass=`[*The password for accessing the samlKeystore.jks*{: style="color: red"}]
* `permission.security.dir=`file:////var/lib/tomcat7/resources/security
* `permission.security.saml.keystore.cert=`[*The name of the private key that was added to the samlKeystore.jks*{: style="color: red"}]
* `permission.security.saml.alias=`[*The Service Provider name for the Permissions component*{: style="color: red"}]
* `oauth.tsb.client=`[*The OAuth client name for the TestSpecBank component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components*{: style="color: red"}]
* `oauth.access.url=`https://[*FQDN or IP address of the OpenAM server*{: style="color: red"}]/auth/oauth2/access_token?realm=/sbac
* `oauth.tsb.client.secret=`[*Password for OAuth client used for TestSpecBank.  Starting value is sbac12345*{: style="color: red"}]
* `permission.oauth.resource.client.secret=`[*Password for OAuth client used for Permissions.  Starting value is sbac12345*{: style="color: red"}]
* `permission.oauth.resource.client.id=`[*The OAuth client name for the Permissions component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components*{: style="color: red"}]
* `permission.oauth.checktoken.endpoint=`https://[*FQDN or IP address of the OpenAM server*{: style="color: red"}]/auth/oauth2/tokeninfo?realm=/sbac

* Example ProgMan properties for Permissions:

~~~~
datasource.url=jdbc:mysql://54.191.75.254:3306/permissions_db
datasource.username=remoteuser
datasource.password=[redacted]
datasource.driverClassName=com.mysql.jdbc.Driver
datasource.minPoolSize=5
datasource.acquireIncrement=5
datasource.maxPoolSize=20
datasource.checkoutTimeout=60000
datasource.maxConnectionAge=0
datasource.acquireRetryAttempts=5
permission.uri=http://54.213.111.234:8080/rest
permission.security.profile=Development
component.name=Permissions
permission.security.idp=https://sso-deployment.sbtds.org/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac
permission.webapp.saml.metadata.filename=perm-saml-sp.xml
permission.security.saml.keystore.pass=[redacted]
permission.security.dir=file:////var/lib/tomcat7/resources/security
permission.security.saml.keystore.cert=perm-saml-sp
permission.security.saml.alias=permissions_web
oauth.tsb.client=tsb
oauth.access.url=https://sso-deployment.sbtds.org/auth/oauth2/access_token?realm=/sbac
oauth.tsb.client.secret=[redacted]
permission.oauth.resource.client.secret=[redacted]
permission.oauth.resource.client.id=pm
permission.oauth.checktoken.endpoint=https://sso-deployment.sbtds.org/auth/oauth2/tokeninfo?realm=/sbac
~~~~

### Deploy Permissions Components

#### Configure Tomcat
* Stop the Tomcat service:
  * `sudo service tomcat7 stop`

{% include checklist/tomcat_java_opts.md %}

* Example:

~~~~
JAVA_OPTS="-Djava.awt.headless=true\
 -XX:+UseConcMarkSweepGC\
 -Xms512m\
 -Xmx1024m\
 -XX:PermSize=512m\
 -XX:MaxPermSize=512m\
 -DSB11_CONFIG_DIR=$CATALINA_BASE/resources\
 -Dspring.profiles.active=progman.client.impl.integration,mna.client.null\
 -Dprogman.baseUri=http://52.34.140.123:8080/rest/\
 -Dprogman.locator=permissions,Development"
~~~~

* Create a directory for the Permissions log files:
  * `sudo mkdir -p /usr/share/tomcat7/logs/permission`
  * `sudo chown -R tomcat7:tomcat7 /usr/share/tomcat7/logs/permission`
* ***OPTIONAL:***  Create link in the Tomcat log directory to the Permissions component log file:
  * `sudo ln -s /usr/share/tomcat7/logs/permission/permission.log /var/lib/tomcat7/logs/permission.log`

{% include checklist/tomcat_install_mysql_connector.md %}

#### Download War File
* Download the latest `.war` file for the Permissions Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://bitbucket.org/fwsbac/permissions_release/downloads/permissions-R01.00.38.war -O /var/lib/tomcat7/webapps/ROOT.war`

{% include checklist/tomcat_pm_client_security_props.md %}

* Start Tomcat to expand the deployed `ROOT.war`:
  * `sudo service tomcat7 start`

{% include checklist/saml_setup.md %}

{% include checklist/saml_registration.md %}

## Update ProgMan Properties Configuration
* Log into the ProgMan server
* Update the `/var/lib/tomcat7/resources/progman/progman-bootstrap.properties` to point to the Permissions instance:
  * `permission.uri=http://`[*FQDN or IP address of Permissions application*{: style="color: #f00;"}]`/rest`

[back to Deployment Checklists](index.html)