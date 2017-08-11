---
title: TDS Database Installation Checklist
permalink: "deployment/checklist/tdsdb.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide databases to support TDS |
| Communicates With | Proctor<br>Student |
| Repository Location | SQL: [https://github.com/SmarterApp/TDS_TestDeliverySystemDataAccess/tree/master/tds-dll-schemas/src/main/resources/sql/MYSQL](https://github.com/SmarterApp/TDS_TestDeliverySystemDataAccess/tree/master/tds-dll-schemas/src/main/resources/sql/MYSQL){:target="_blank"}<br>
Utilities for creating databases: [https://github.com/SmarterApp/TDS_Build/tree/master/database/tds](https://github.com/SmarterApp/TDS_Build/tree/master/database/tds)  |
| Additional Documentation | Not Applicable |

# Instructions

{% include checklist/rds_setup.md %}

**NOTE:** When creating the TDS databases, the RDS instance must be accessible remotely.  A remote computer must be able to connect to it with a user account that has sufficient privileges to create schemas and objects within those schemas.

### Create Test Delivery System (TDS) Databases
* If not done already, install [git](https://git-scm.com/) on the machine that will be responsible for creating the TDS databases
* If not done already, clone the `TDS_Build` and `TDS_TestDeliverySystemDataAccess` repositories from GitHub:
  * `git clone https://github.com/SmarterApp/TDS_Build.git`
  * `git clone https://github.com/SmarterApp/TDS_TestDeliverySystemDataAccess.git`
* ***NOTE:*** When cloning `TDS_Build` and `TDS_TestDeliverySystemDataAccess`, they should be "siblings" at the same directory level.  For example, if both repositories are cloned in the `ubuntu` user's home directory, the directory will look like this:

~~~~
ubuntu@tds-db-deploy:~$ ls -alh
total 48K
drwxr-xr-x  6 ubuntu ubuntu 4.0K May  2 17:57 .
drwxr-xr-x  3 root   root   4.0K May  2 17:22 ..
-rw-r--r--  1 ubuntu ubuntu  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 ubuntu ubuntu 3.6K May  2 17:24 .bashrc
drwx------  2 ubuntu ubuntu 4.0K May  2 17:23 .cache
-rw-------  1 ubuntu ubuntu  135 May  2 17:56 .mysql_history
-rw-r--r--  1 ubuntu ubuntu  675 Apr  9  2014 .profile
drwx------  2 ubuntu ubuntu 4.0K May  2 17:22 .ssh
drwxrwxr-x 13 ubuntu ubuntu 4.0K May  2 17:58 TDS_Build
drwxrwxr-x 12 ubuntu ubuntu 4.0K May  2 17:57 TDS_TestDeliverySystemDataAccess
-rw-------  1 ubuntu ubuntu 4.2K May  2 17:54 .viminfo
~~~~

* Navigate to the `TDS_Build/database/tds` directory:
  * `cd TDS_Build/database/tds`
* Update the `db-schema-setup.sh` script to use the correct user name and password (lines 20 and 21):
  * `HOST=[the host name or IP address of the database server that will host the TDS databases]`
  * `PORT=[the port on which the database server is listening]`
  * `USER=[the user name with sufficient privileges to create schemas and objects within those schemas]`
  * `PW=[the MySQL user's password]`
* If necessary, update the port and hostame (lines 18 and 19)
* Make the `db-schema-setup.sh` executable:
  * `sudo chmod u+x db-schema-setup.sh`
* Run the `db-schema-setup.sh` script to create the TDS database schema and load it with seed data:
  * `./db-schema-setup.sh`

#### Verify the TDS Database Schemas Were Created and Populated With Seed Data
* Log into the RDS instance:

<code>mysql -u [<span class="placeholder">the user name with sufficient privileges to create schemas and objects within those schemas</span>] -p -h [<span class="placeholder">the host name or IP address of the database server that will host the TDS databases</span>] -P [<span class="placeholder">the port on which the database server is listening</span>]</code>

* Execute the following query:
  * `show databases;`
* Output should appear as follows:

~~~~
+--------------------+
| Database           |
+--------------------+
| information_schema |
| archive            |
| configs            |
| itembank           |
| mysql              |
| performance_schema |
| session            |
+--------------------+
~~~~

* Change to the `archive` database:
  * `use archive;`
* List the tables in the database:
  * `show tables;`

~~~~
+-----------------------------+
| Tables_in_archive           |
+-----------------------------+
| _dblatency                  |
| _dblatencyarchive           |
| _dblatencyreports           |
| auditaccommodations         |
| opportunityaudit            |
| opportunityclient           |
| serverlatency               |
| serverlatencyarchive        |
| sessionaudit                |
| systemclient                |
| systemerrors                |
| systemerrorsarchive         |
| testopportunityscores_audit |
+-----------------------------+
13 rows in set (0.00 sec)
~~~~

* Change to the `configs` database:
  * `use configs;`
* List the tables in the database:
  * `show tables;`

~~~~
+------------------------------------+
| Tables_in_configs                  |
+------------------------------------+
| __accommodationcache               |
| __appmessagecontexts               |
| __appmessages                      |
| __cachedaccomdepends               |
| __cachedaccommodations             |
| _messageid                         |
| client                             |
| client_accommodationfamily         |
| client_accommodations              |
| client_allowips                    |
| client_externs                     |
| client_fieldtestpriority           |
| client_forbiddenappsexcludeschools |
| client_forbiddenappslist           |
| client_grade                       |
| client_itemscoringconfig           |
| client_language                    |
| client_message                     |
| client_messagearchive              |
| client_messagetranslation          |
| client_messagetranslationaudit     |
| client_pilotschools                |
| client_rtsroles                    |
| client_segmentproperties           |
| client_server                      |
| client_subject                     |
| client_systemflags                 |
| client_tds_rtsattribute            |
| client_tds_rtsattributevalues      |
| client_test_itemconstraint         |
| client_test_itemtypes              |
| client_testeeattribute             |
| client_testeerelationshipattribute |
| client_testeligibility             |
| client_testformproperties          |
| client_testgrades                  |
| client_testkey                     |
| client_testmode                    |
| client_testprerequisite            |
| client_testproperties              |
| client_testrtsspecs                |
| client_testscorefeatures           |
| client_testtool                    |
| client_testtoolrule                |
| client_testtooltype                |
| client_testwindow                  |
| client_timelimits                  |
| client_timewindow                  |
| client_tooldependencies            |
| client_toolusage                   |
| client_voicepack                   |
| geo_clientapplication              |
| geo_database                       |
| rts_role                           |
| statuscodes                        |
| system_applicationsettings         |
| system_browserwhitelist            |
| system_networkdiagnostics          |
| systemerrors                       |
| tds_application                    |
| tds_applicationsettings            |
| tds_browserwhitelist               |
| tds_clientaccommodationtype        |
| tds_clientaccommodationvalue       |
| tds_configtype                     |
| tds_coremessageobject              |
| tds_coremessageuser                |
| tds_fieldtestpriority              |
| tds_role                           |
| tds_systemflags                    |
| tds_testeeattribute                |
| tds_testeerelationshipattribute    |
| tds_testproperties                 |
| tds_testtool                       |
| tds_testtoolrule                   |
| tds_testtooltype                   |
+------------------------------------+
76 rows in set (0.00 sec)
~~~~

* Change to the `itembank` database:
  * `use itembank;`
* List the tables in the database:
  * `show tables;`

~~~~
+---------------------------------------+
| Tables_in_itembank                    |
+---------------------------------------+
| _dblatency                            |
| _synonyms                             |
| _sys_formtestitems                    |
| _testupdate                           |
| aa_itemcl                             |
| affinitygroup                         |
| affinitygroupitem                     |
| alloweditemprops                      |
| configsloaded                         |
| importitemcohorts                     |
| importtestcohorts                     |
| itemmeasurementparameter              |
| itemscoredimension                    |
| loader_errors                         |
| loader_itemscoredimension             |
| loader_itemscoredimensionproperties   |
| loader_measurementparameter           |
| loader_segment                        |
| loader_segmentblueprint               |
| loader_segmentform                    |
| loader_segmentitemselectionproperties |
| loader_segmentpool                    |
| loader_segmentpoolgroupitem           |
| loader_segmentpoolpassageref          |
| loader_setofitemstrands               |
| loader_testblueprint                  |
| loader_testformgroupitems             |
| loader_testformitemgroup              |
| loader_testformpartition              |
| loader_testformproperties             |
| loader_testforms                      |
| loader_testitem                       |
| loader_testitempoolproperties         |
| loader_testitemrefs                   |
| loader_testpackage                    |
| loader_testpackageproperties          |
| loader_testpassages                   |
| loader_testpoolproperties             |
| measurementmodel                      |
| measurementparameter                  |
| performancelevels                     |
| projects                              |
| setoftestgrades                       |
| tbladminstimulus                      |
| tbladminstrand                        |
| tbladminsubjecttestpackage            |
| tblalternatetest                      |
| tblclient                             |
| tblitem                               |
| tblitembank                           |
| tblitemgroup                          |
| tblitemprops                          |
| tblitemselectionparm                  |
| tblsetofadminitems                    |
| tblsetofadminsubjects                 |
| tblsetofitemstimuli                   |
| tblsetofitemstrands                   |
| tblstimulus                           |
| tblstrand                             |
| tblsubject                            |
| tbltestadmin                          |
| tbltestpackage                        |
| testcohort                            |
| testform                              |
| testformitem                          |
+---------------------------------------+
65 rows in set (0.00 sec)
~~~~

* Change to the `session` database:
  * `use session;`
* List the tables in the database:
  * `show tables;`

~~~~
+-----------------------------------+
| Tables_in_session                 |
+-----------------------------------+
| _anonymoustestee                  |
| _externs                          |
| _maxtestopps                      |
| _missingmessages                  |
| _sb_errorlog                      |
| _sb_messagehandler                |
| _sb_messages                      |
| _sb_messagesarchive               |
| _sitelatency                      |
| _synonyms                         |
| adminevent                        |
| admineventitems                   |
| admineventopportunities           |
| alertmessages                     |
| client_os                         |
| client_reportingid                |
| client_sessionid                  |
| clientlatency                     |
| clientlatencyarchive              |
| externs                           |
| ft_groupsamples                   |
| ft_opportunityitem                |
| geo_clientsystem                  |
| geo_session                       |
| itemdistribution                  |
| loadtest_testee                   |
| qareportqueue                     |
| qc_validationexception            |
| r_abnormallogins                  |
| r_blueprintreport                 |
| r_geolatencyreport                |
| r_hourlygeolatencytable           |
| r_hourlyusers                     |
| r_participationcountstable        |
| r_proctorpackage                  |
| r_schoolparticipationreport       |
| r_studentkeyid                    |
| r_studentpackage                  |
| r_studentpackagedetails           |
| r_testcounts                      |
| rtsschoolgrades                   |
| session                           |
| sessiontests                      |
| setofproctoralertmessages         |
| sim_defaultitemselectionparameter |
| sim_itemgroup                     |
| sim_itemselectionparameter        |
| sim_segment                       |
| sim_segmentcontentlevel           |
| sim_segmentitem                   |
| sim_sessiontestpackage            |
| sim_user                          |
| sim_userclient                    |
| simp_itemgroup                    |
| simp_itemselectionparameter       |
| simp_segment                      |
| simp_segmentcontentlevel          |
| simp_segmentitem                  |
| simp_session                      |
| simp_sessiontests                 |
| sirve_audit                       |
| sirve_session                     |
| statuscodes                       |
| tblclsclientsessionstatus         |
| tbluser                           |
| testeeaccommodations              |
| testeeattribute                   |
| testeecomment                     |
| testeehistory                     |
| testeeitemhistory                 |
| testeerelationship                |
| testeeresponse                    |
| testeeresponsearchive             |
| testeeresponseaudit               |
| testeeresponsescore               |
| testoppabilityestimate            |
| testopportunity                   |
| testopportunity_readonly          |
| testopportunitycontentcounts      |
| testopportunityscores             |
| testopportunitysegment            |
| testopportunitysegmentcounts      |
| testopprequest                    |
| testopptoolsused                  |
| timelimits                        |
+-----------------------------------+
85 rows in set (0.00 sec)
~~~~

* Change to the `itembank` database:
  * `use itembank;`
* List the assessments that have been loaded:
  * `select _key, testid from tblsetofadminsubjects;`

~~~~
+------------------------------------------------+------------------------------+
| _key                                           | testid                       |
+------------------------------------------------+------------------------------+
| (SBAC_PT)SBAC ELA 3-ELA-3-Spring-2014-2015     | SBAC ELA 3-ELA-3             |
| (SBAC_PT)SBAC ELA 6-ELA-6-Spring-2014-2015     | SBAC ELA 6-ELA-6             |
| (SBAC_PT)SBAC ELA HS-ELA-10-Spring-2014-2015   | SBAC ELA HS-ELA-10           |
| (SBAC_PT)SBAC Math 3-MATH-3-Spring-2014-2015   | SBAC Math 3-MATH-3           |
| (SBAC_PT)SBAC Math 6-MATH-6-Spring-2014-2015   | SBAC Math 6-MATH-6           |
| (SBAC_PT)SBAC Math 7-MATH-7-Spring-2014-2015   | SBAC Math 7-MATH-7           |
| (SBAC_PT)SBAC Math HS-MATH-10-Spring-2014-2015 | SBAC Math HS-MATH-10         |
| (SBAC_PT)SBAC-Student Help-11-Spring-2014-2015 | SBACTraining-Student Help-11 |
+------------------------------------------------+------------------------------+
8 rows in set (0.00 sec)
~~~~

***NOTE:*** The tests loaded into the `itembank` database come from the [Practice and Training Tests Package](ftp://ftps.smarterbalanced.org/~sbacpublic/Public/PracticeAndTrainingTests/2015-08-28_TrainingTestPackagesAndContent.zip){:target="_blank"}.
