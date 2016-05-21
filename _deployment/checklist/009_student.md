---
title: Student Installation Checklist
permalink: "deployment/checklist/student.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide interface for students to take assessments |
| Communicates With | ProgMan, ART |
| Repository Location | [https://bitbucket.org/sbacoss/student_release](https://bitbucket.org/sbacoss/student_release |
| Additional Documentation | [student-progman-config.txt](https://bitbucket.org/sbacoss/student_release/src/a5de012d932d58a2cf1e29c06fd8047fcbce1a00/Documents/Installation/student-progman-config.txt?at=default) |

***NOTE:***{: style="color: #f00"} *The Student and Proctor applications are deployed to the same server.  That is, a single AWS instance will host the Student and Proctor applications.*{: style="color: #04384e"}

# Instructions - Deploy Student to AWS Instance That Already Hosts Proctor

## Student Application Setup

### Configure Student in ProgMan
{% include checklist/progman_config.md %}

***IMPORTANT:***  When deploying the Student application to an AWS instance that hosts the Proctor application, choose the ProgMan entry that is already in use by Proctor.

* Add the following properties to the ProgMan entry for the Proctor application:
  * `student.ClientName=`[*Name of client that "owns" assessments in database.  Set to **SBAC_PT** for training tests, set to **SBAC** for all others*{: style="color: red"}]
  * `student.oauth.resource.client.secret=`[*Password for OAuth client used for Proctor.  Starting value is **sbac12345***{: style="color: red"}]
  * `student.StateCode=`[*The name of the STATE-level Client in ProgMan*{: style="color: red"}]
  * `student.TestRegistrationApplicationUrl=`http://[*FQDN or IP address of ART component*{: style="color: red"}]/rest
  * `performance.datasource.maxPoolSize=`200
  * `performance.datasource.minPoolSize=`60

### Deploy Student Component

#### Create Student Log File Directories
* Create a directory for Student log file:
  * `sudo mkdir -p /usr/share/tomcat7/logs/student`
  * `sudo chown -R tomcat7:tomcat7 /usr/share/tomcat7/logs`
* ***OPTIONAL:***  Create links in the Tomcat log directory to the Web Application log file:
  * `sudo ln -s /usr/share/tomcat7/logs/student/student.log /var/lib/tomcat7/logs/student.log`

#### Download War Files
* Download the latest `.war` file for the Student Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://bitbucket.org/fwsbac/tds_release/downloads/`[*Latest .war file*{: style="color: #f00;"}]` -O /var/lib/tomcat7/webapps/student.war`
  * Exmaple:
    * `sudo wget https://bitbucket.org/fwsbac/tds_release/downloads/`<span class="placeholder-example">student-1.0.1-BUILD-SNAPSHOT.war</span>` -O /var/lib/tomcat7/webapps/student.war`

#### Update the pm-client-security.properties File
* Add the following properties to the Proctor's `pm-client-security.properties`:
  * `oauth.testreg.client.id=`[*The OAuth client name for the ART component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components*{: style="color: red"}]
  * `oauth.testreg.client=`[*The OAuth client name for the ART component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components*{: style="color: red"}]
  * `oauth.testreg.client.secret=`[*Password for OAuth client used for ART.  Starting value is sbac12345*{: style="color: red"}]
  * `oauth.testreg.client.granttype=`password
  * `oauth.testreg.username=`[*User account in OpenDJ, e.g. the Prime User account*{: style="color: red"}]
  * `oauth.testreg.password=`[*Password for OpenDJ user account*{: style="color: red"}]
  * `tds.iris.EncryptionKey=`[*The encryption key used by IRiS.  Must be at least 24 characters*{: style="color: red"}]

* An example of the `pm-client-security.properties` file configured for the Student and Proctor application:

<div class="highlighter-rouge">
<pre class="highlight">
<code>#security props
oauth.access.url=https://<span class="placeholder-example">sso-dev.sbtds.org</span>/auth/oauth2/access_token?realm=/sbac
pm.oauth.client.id=<span class="placeholder-example">pm</span>
pm.oauth.client.secret=<span class="placeholder-example">[redacted]</span>
pm.oauth.batch.account=<span class="placeholder-example">prime.user@example.com</span>
pm.oauth.batch.password=<span class="placeholder-example">[redacted]</span>
oauth.testreg.client.id=<span class="placeholder-example">testreg</span>
# This line is in here because the readme for tds_release cites oauth.testreg.client.  On the other hand, the student_release
# readme states oauth.testreg.client.id.
# TODO:  Find out if oauth.testreg.client is really needed by tds_release or if it can use the same property as what's cited in the student_release readme
oauth.testreg.client=<span class="placeholder-example">testreg</span>
oauth.testreg.client.secret=<span class="placeholder-example">[redacted]</span>
oauth.testreg.client.granttype=<span class="placeholder-example">password</span>
oauth.testreg.username=<span class="placeholder-example">prime.user@example.com</span>
oauth.testreg.password=<span class="placeholder-example">[redacted]</span>
tds.iris.EncryptionKey=Thisisanincrediblylongkeythatiscertainlylongerthantwentyfourcharacters</code>
</pre>
</div>

[back to Deployment Checklists](index.html)