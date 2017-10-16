---
title: Item Scoring Installation Checklist
permalink: "deployment/checklist/student_3x/itemscoring.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "3x"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Scores test items |
| Communicates With | Student<br>TIS |
| Repository Location | [https://github.com/SmarterApp/TDS_ItemScoring](https://github.com/SmarterApp/TDS_ItemScoring){:target="_blank"} |
| Additional Documentation | [README](https://github.com/SmarterApp/TDS_ItemScoring/blob/master/README.md){:target="_blank"}<br>[Running Item Scoring Engine](https://github.com/SmarterApp/TDS_ItemScoring/blob/master/docs/Running%20Item%20Scoring%20Engine.docx){:target="_blank"} |

***NOTE:***{: style="color: #f00;"} *The Item Scoring component is typically deployed to the same application server that hosts the Student and Proctor applications.  That is, a single AWS instance will host the Student and Proctor applications.*{: style="color: #04384e;"}

# Instructions - Deploy Item Scoring to AWS Instance That Already Hosts Student and Proctor

## Deploy the Item Scoring Component

### Create Log Directory
* Create a directory for the Item Scoring component's log file(s):
  * `sudo mkdir -p /usr/share/tomcat7/logs/itemscoring/`
  * `sudo chown tomcat7:tomcat7 /usr/share/tomcat7/logs/itemscoring/`
  * ***OPTIONAL:***  Create links in the Tomcat log directory to the Web Application log file:
    * `sudo ln -s /usr/share/tomcat7/logs/itemscoring/itemscoring.log /var/lib/tomcat7/logs/itemscoring.log`

### Deploy the Item Scoring Component
* Stop the Tomcat server:
  * `sudo service tomcat7 stop`
* Download the latest `.war` file for the Item Scoring Component into the Tomcat server's `webapps` directory:
  * `sudo wget https://github.com/SmarterApp/TDS_ItemScoring/releases/download/1.1.0/item-scoring-service-1.1.0.war -O var/lib/tomcat7/webapps/itemscoring.war`

### Update Item Scoring Configuration in the Itembank Database
* Connect to the MySQL server that supports the TDS Components
* Execute the following SQL statements:

~~~~
SET SQL_SAFE_UPDATES = 0;
START TRANSACTION;
UPDATE configs.client_itemscoringconfig
SET serverurl = 'http://localhost:8080/itemscoring/Scoring/ItemScoring'
WHERE serverurl != 'http://localhost:8080/itemscoring/Scoring/ItemScoring';
COMMIT;
SET SQL_SAFE_UPDATES = 1;

SET SQL_SAFE_UPDATES = 0;
START TRANSACTION;
UPDATE configs.client_itemscoringconfig
SET clientname = 'SBAC_PT'
WHERE clientname = 'SBAC';
COMMIT;
SET SQL_SAFE_UPDATES = 1;
~~~~

***IMPORTANT:***{: style="color: #f00;"} *If your deployment is using an environment designation other than **Development**, run the `UPDATE` statement below*{: style="background-color: #ff0;"}

<div class="highlighter-rouge">
<pre class="highlight">
<code>SET SQL_SAFE_UPDATES = 0;
START TRANSACTION;
UPDATE configs.client_itemscoringconfig
SET environment = '[<span class="placeholder">name of environment set in the <strong>progman.locator</strong> of <strong>JAVA_OPTS</strong> contained within the <strong>/etc/default/tomcat7</strong> file.</span>]';
COMMIT;
SET SQL_SAFE_UPDATES = 1;</code>
</pre>
</div>

* Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>SET SQL_SAFE_UPDATES = 0;
START TRANSACTION;
UPDATE configs.client_itemscoringconfig
SET environment = '<span class="placeholder-example">Staging</span>';
COMMIT;
SET SQL_SAFE_UPDATES = 1;</code>
</pre>
</div>

* Start Tomcat:
  * `sudo service tomcat7 start`

[back to Deployment Checklists](index.html)
