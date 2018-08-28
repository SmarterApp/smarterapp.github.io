---
title: Support Tool Installation Checklist
permalink: "/deployment/checklist/student_4x/support_tool.html"
layout: "document"
categories: ["deployment", "checklist", "tds", "4x"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide interface for loading test packages and running scoring validation reports |
| Communicates With | OpenAM<br>Permissions<br>TIS<br>ERT |
| Repository Location | [https://github.com/SmarterApp/TDS_SupportTool](https://github.com/SmarterApp/TDS_SupportTool){:target="_blank"} |
| Additional Documentation | [README](https://github.com/SmarterApp/TDS_SupportTool/blob/develop/README.md)|


## Instructions

### Create the Support Tool Database
* Create the following database on the MongoDB Server:
  * `support-tool`

### EC2 Container Service
 Create [EC2 Container Repositories](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_Console_Repositories.html) to host Proctor Docker images (separate one for each) containing the SAML and security information.

### Permission Settings

If you have already run db-perm-schema-setup.sh in a previous step, all the permissions that you need
should already be configured. However, if you are working with an existing TDS cluster and just need to
upgrade the permissions to include the ones used by Support Tool, you will have to run the Support Tool
script separately:

* Navigate to the `TDS_Build\database\permissions` directory:
  * `cd TDS_Build\database\permissions`

* Execute the Support Tool script with the proper parameters for the TDS MySQL instance:
  * mysql --host="$HOST" --port="$PORT" --user="$USER" --password="$PW" --database=permissions_db < db-perm-seed-data-supporttool.sql

* Log into MySQL:
  * `mysql -u root -p`
* Execute the following command:
  * `use permissions_db;`
* Execute the folowing query:
  * `select * from component;`
* Output should include: Support Tool
* Execute the folowing query:
  * `select * from role;`
* Output should include: Support Tool Admin, Support Tool Validator Admin, and Support Tool Test Package Admin
* Execute the folowing query:
  * `select * from permission;`
* Output should include: Support Tool Administration, Scoring Validator, and Test Package Loader

### OpenAM Updates
* In order to get SAML working and enable logging into SupportTool via the Single Sign-On, you will need to add Support Tool as a Service Provider in OpenAM.
* Log into the OpenAM administration console located at: 
	* https://[*Base OpenAM URL*{: style="color: #f00;"}]/auth/console?realm=/
* 	The default username is `amadmin`
*  Click on the `Federation` tab
*  In the `Entity Providers` section of the page, click `Import Entity...`
*  Enter the following:
	* Change Realm to `/sbac`
	* Select `File` for the `Where does the metadata file reside?:` option
	* Upload the `sp.xml` file
	* Ignore the second option for uploading a file or URL, leave it blank
	* Click `'Ok`
* Click on the `sbac` link within the `Circle of Trust` section of the page
* Click the `Add >` button to move the newly created `Support Tool` item to the `Selected` list
* Click `Save`  

[back to Deployment Checklists](index.html)
