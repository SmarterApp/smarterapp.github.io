---
title: Student Report Processor (SRP) Installation Checklist
permalink: "deployment/checklist/srp.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Background processor that submits completed tests to TIS |
| Communicates With | Student<br>TIS |
| Repository Location | [https://github.com/SmarterApp/TDS_StudentReportProcessor](https://github.com/SmarterApp/TDS_StudentReportProcessor){:target="_blank"} |
| Additional Documentation | [README](https://github.com/SmarterApp/TDS_StudentReportProcessor/blob/master/README.md){:target="_blank"} |

***NOTE:***{: style="color: #f00"} *The Student Report Processor is typically deployed to the same application server that hosts the Student and Proctor applications.  That is, a single AWS instance will host the Student and Proctor applications.*{: style="color: #04384e"}

# Instructions - Deploy Student Report Processor to AWS Instance That Already Hosts Student and Proctor

## Deploy the Student Report Processor JAR

### Create Directory Structure
* Create a directory structure for the SRP `.jar` and associated files:
  * `mkdir -p `[*Path/to/SRP/jar/and/files*{: style="color: #f00;"}]
  * **NOTE:** Depending on where the directory is created, `sudo` may be required to execute the `mkdir` command
  * Example:
    * `sudo mkdir -p `<span class="placeholder-example">/opt/sbtds/srp/logs</span>

### Download the SRP .jar
* Download the latest `.jar` file for the SRP Component into the desired directory:
  * `sudo wget https://github.com/SmarterApp/TDS_StudentReportProcessor/releases/download/1.0.1/StudentReportProcessor-1.0.1.jar -O `[*Path to where the SRP .jar should be deployed*{: style="color: #f00;"}]`
    * Example: `sudo wget https://github.com/SmarterApp/TDS_StudentReportProcessor/releases/download/1.0.1/StudentReportProcessor-1.0.1.jar` -O `<span class="placeholder-example">/opt/sbtds/srp</span>`StudentReportProcessor-0.0.1-SNAPSHOT.jar`

### Configure the SRP
***IMPORTANT:***{: style="color: #f00;"} *The SRP does not have any entries in ProgMan.  The SRP is configured entirely by settings passed into the `JAVA_OPTS`.*{: style="background-color: #ff0;"}

* Create a script (e.g. <span class="placeholder-example">/opt/sbtds/srp/start-srp.sh</span>) that will start up the SRP application:

<div class="highlighter-rouge">
<pre class="highlight">
<code>### create shell script to run app - start-srp.sh
# had to change the classpath values for the StudentReportProcessor jar
#   and pointed the mysql-connector jar to the version deployed with student
#######

#!/usr/bin/env bash

/usr/bin/java \
    -DreportingDLL.tisUrl="http://[<span class="placeholder">FQDN or IP address for TIS</span>]/api/testresult" \
    -DreportingDLL.tisStatusCallbackUrl="http://[<span class="placeholder">FQDN or IP address for TIS</span>]/student/Pages/API/tisReply" \
    -Doauth.tis.client.id="[<span class="placeholder">The OAuth client name for the ART component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components</span>]" \
    -Doauth.tis.client.secret="[<span class="placeholder">Password for OAuth client used for ART.  Starting value is sbac12345</span>] \
    -Doauth.tis.username="[<span class="placeholder">User account in OpenDJ.  <strong>NOTE:</strong> This user must belong to the <strong>TIS Admin</strong> role (mentioned below)</span>] \
    -Doauth.tis.password="[<span class="placeholder">Password for account in OpenDJ</span>] \
    -Doauth.access.url="https://[<span class="placeholder">FQDN or IP address for OpenAM</span>]/auth/oauth2/access_token?realm=/sbac" \
    -Dlogfilename="[<span class="placeholder">Path to where SRP should write its log entries</span>]" \
    -Djdbc.driver="com.mysql.jdbc.Driver" \
    -Djdbc.url="jdbc:mysql://[<span class="placeholder">FQDN or IP address to MySQL database server hosting the session database</span>]/session" \
    -Djdbc.userName="[<span class="placeholder">MySQL user account that has access to read/write in the TDS databases</span>]" \
    -Djdbc.password="[<span class="placeholder">password for MySQL user account that has access to read/write in the TDS databases</span>]" \
    -DDBDialect="MYSQL" \
    -classpath ".:[<span class="placeholder">/Path/to/where/SRP/is/deployed</span>]/StudentReportProcessor-0.0.1-SNAPSHOT.jar:[<span class="placeholder">/Path/to/mysqlconnector.jar</span>]/mysql-connector-java-5.1.26.jar" org.opentestsystem.delivery.studentreportprocessor.StudentReportProcessor</code>
</pre>
</div>

* Exmaple:

<div class="highlighter-rouge">
<pre class="highlight">
<code>### create shell script to run app - start-srp.sh
# had to change the classpath values for the StudentReportProcessor jar
#   and pointed the mysql-connector jar to the version deployed with student
#######

#!/usr/bin/env bash

/usr/bin/java \
    -DreportingDLL.tisUrl="http://<span class="placeholder-example">54.200.55.254:8080</span>/api/testresult" \
    -DreportingDLL.tisStatusCallbackUrl="http://<span class="placeholder-example">54.200.55.254:8080</span>/student/Pages/API/tisReply" \
    -Doauth.tis.client.id="<span class="placeholder-example">pm</span>" \
    -Doauth.tis.client.secret="<span class="placeholder-example">sbac12345</span>" \
    -Doauth.tis.username="<span class="placeholder-example">ca.admin@example.com</span>" \
    -Doauth.tis.password="<span class="placeholder-example">[redacted]</span>" \
    -Doauth.access.url="https://<span class="placeholder-example">sso-deployment.sbtds.org</span>/auth/oauth2/access_token?realm=/sbac" \
    -Dlogfilename="<span class="placeholder-example">/home/ubuntu/deploy/new/srp</span>" \
    -Djdbc.driver="com.mysql.jdbc.Driver" \
    -Djdbc.url="jdbc:mysql://<span class="placeholder-example">52.33.126.67</span>/session" \
    -Djdbc.userName="<span class="placeholder-example">remoteuser</span>" \
    -Djdbc.password="<span class="placeholder-example">[redacted]</span>" \
    -DDBDialect="MYSQL" \
    -classpath ".:<span class="placeholder-example">/home/ubuntu/deploy/new</span>/StudentReportProcessor-0.0.1-SNAPSHOT.jar:<span class="placeholder-example">/var/lib/tomcat7/webapps/student/WEB-INF/lib</span>/mysql-connector-java-5.1.26.jar" org.opentestsystem.delivery.studentreportprocessor.StudentReportProcessor</code>
</pre>
</div>
* Change ownership of the directory structure where the SRP `.jar` and associated files are deployed:
  * `sudo chown -R `[*The Linux user account that should run the SRP process*{: style="color: #f00;"}]` `[*The directory containing the SRP files*{: style="color: #f00;"}]
    * Example: `sudo chown -R `<span class="placeholder-example">ubuntu:ubuntu</span>` `<span class="placeholder-example">/opt/sbtds</span>
* Make the script executable:
  * `sudo chmod u+x `[*Path/to/script/script-name*{: style="color: #f00;"}]
    * Example: `sudo chmod u+x `<span class="placeholder-example">/opt/sbtds/srp/start-srp.sh</span>

### Update Permissions to Support SRP
* Log into the [Permissions](permissions.html) application
* Verify the **TIS Admin** Role exists:
  * Click on **Manage Roles**
  * If a **TIS Admin** (*case sensitive*) role does not exist, create it by clicking **Add New Role** button and creating a **TIS Admin** role
* **NOTE:** The **TIS Admin** role does not need to have any permissions associated to it
* Verify the **TIS Admin** role is associated to the following Tenant levels:
  * **CLIENT**
  * **STATE**
  * **DISTRICT**
  * **INSTITUTION**

### Add a User Account to the TIS Admin Role
* Log into the [ART](art.html) application
* Navigate to **Create/Modify User**
* Create a new or choose an existing user account for use with the SRP component
* Associate the chosen user account to the **TIS Admin** role

### Start the SRP Component
* Run the SRP script as a background process:
  * `sudo nohup `<span class="placeholder-example">/opt/sbtds/srp/start-srp.sh</span>` &`

[back to Deployment Checklists](index.html)