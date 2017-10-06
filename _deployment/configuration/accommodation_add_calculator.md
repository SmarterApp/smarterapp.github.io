---
title: How to Add a Calculator Accommodation
permalink: "deployment/configuration/accommodation_add_calculator.html"
layout: "document"
categories: ["student", "accommodations"]
---

## Overview
The calculator user interface (UI) is provided by [Desmos](https://www.desmos.com/).  There are three types of calculator accommodation available:

* **Basic:**  Also known as the four-function calculator, this calculator provides support for addition, subtraction, multiplication and division
* **Scientific:**  Allows for a much wider range of calculator functionality, allowing students to solve complex math problems
* **Graphing:**  In addition to supporting the features of the scientific calculator, this UI can be used to plot graphs.  Also provides support for linear regression algebra.

The calculator accommodation can be assigned to an assessment as an "assessment-wide" accommodation (that is, the calculator is available in every segment of the assessment) or it can be assigned to a specific segment.

## Creating a Calculator Accommodation
### Pre-requisites
To create an assessment-wide Basic Calculator accommodation, the following information is required:

* **client name:**  The client name to which the accommodation will be assigned.  Typically, this value will be **SBAC** for production or **SBAC_PT** for a practice test environment.
  * Whatever value is chosen, it must be consistent with the assessment to which this calculator accommodation is assigned.  If the assessment is associated with **SBAC**, then this accommodation must also be associated with **SBAC**
* **test identifier:**  The test ID of the assessment to which this calculator accommodation will be assigned to.
  * This value can be found in the `testid` column of the `itembank.tblsetofadminsubjects` table.

### Calculator Creation Steps
To create a calculator accommodation for an assessment, take the following steps:

1.  Determine the `testid` of the assessment that should have a calculator assigned to it
2.  Determine the type of calculator that should be assigned to the Assessment
3.  Copy the SQL for the desired calculator into an editor
4.  Replace the placeholders with appropriate values
5.  Execute the SQL against your database

#### SQL to Create an Assessment-Wide Basic Calculator Accommodation
**NOTE:**  There must be two records per language - one for the calculator code (e.g. TDS_CalcBasic) and another for the disabled calculator code (i.e. `TDS_Calc0`).  The `INSERT`s below insert a dependency for **English (ENU)** and **Spanish (ESN)**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
INSERT INTO `client_testtooltype` (`clientname`,`toolname`,`allowchange`,`tideselectable`,`rtsfieldname`,`isrequired`,`tideselectablebysubject`,`isselectable`,`isvisible`,`studentcontrol`,`tooldescription`,`sortorder`,`dateentered`,`origin`,`source`,`contexttype`,`context`,`dependsontooltype`,`disableonguestsession`,`isfunctional`,`testmode`,`isentrycontrol`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator',1,0,'TDSAcc-Calculator',0,0,1,1,0,NULL,0,NOW(),'TDSCore_Staging_Configs_2013','TDSCore_Staging_Configs_2013','TEST','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','Language',0,1,'ALL',0);

INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_Calc0','None',0,0,'Calculator feature is disabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',1,'TDSCore_Staging_Configs_2012','TDSCore_Staging_Configs_2012','TEST','ALL',NULL);
INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_CalcBasic','Basic',1,0,'Basic Calculator is enabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',1,'TDSCore_Staging_Configs_2012','TDSCore_Staging_Configs_2012','TEST','ALL',NULL);

INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','TEST','Language','ESN',1,'Calculator','TDS_CalcBasic','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',1,'Calculator','TDS_CalcBasic','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ESN',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
</code>
</pre>
</div>

#### SQL to Create an Assessment-Wide Scientific Calculator Accommodation
**NOTE:**  There must be two records per language - one for the calculator code (e.g. TDS_CalcBasic) and another for the disabled calculator code (i.e. `TDS_Calc0`).  The `INSERT`s below insert a dependency for **English (ENU)** and **Spanish (ESN)**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
INSERT INTO `client_testtooltype` (`clientname`,`toolname`,`allowchange`,`tideselectable`,`rtsfieldname`,`isrequired`,`tideselectablebysubject`,`isselectable`,`isvisible`,`studentcontrol`,`tooldescription`,`sortorder`,`dateentered`,`origin`,`source`,`contexttype`,`context`,`dependsontooltype`,`disableonguestsession`,`isfunctional`,`testmode`,`isentrycontrol`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator',1,0,'TDSAcc-Calculator',0,0,1,1,0,NULL,0,NOW(),'TDSCore_Staging_Configs_2013','TDSCore_Staging_Configs_2013','TEST','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','Language',0,1,'ALL',0);

INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_Calc0','None',0,0,'Calculator feature is disabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',-1,'Fairway','Fairway','TEST','ALL',NULL);
INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_CalcSciInv','ScientificInv',1,0,'Scientific Calculator is enabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',4,'Fairway','Fairway','TEST','ALL',NULL);

INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ESN',1,'Calculator','TDS_CalcSciInv','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',1,'Calculator','TDS_CalcSciInv','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ESN',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
</code>
</pre>
</div>

#### SQL to Create an Assessment-Wide Graphing Calculator Accommodation
**NOTE:**  There must be two records per language - one for the calculator code (e.g. TDS_CalcBasic) and another for the disabled calculator code (i.e. `TDS_Calc0`).  The `INSERT`s below insert a dependency for **English (ENU)** and **Spanish (ESN)**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
INSERT INTO `client_testtooltype` (`clientname`,`toolname`,`allowchange`,`tideselectable`,`rtsfieldname`,`isrequired`,`tideselectablebysubject`,`isselectable`,`isvisible`,`studentcontrol`,`tooldescription`,`sortorder`,`dateentered`,`origin`,`source`,`contexttype`,`context`,`dependsontooltype`,`disableonguestsession`,`isfunctional`,`testmode`,`isentrycontrol`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator',1,0,'TDSAcc-Calculator',0,0,1,1,0,NULL,0,NOW(),'TDSCore_Staging_Configs_2013','TDSCore_Staging_Configs_2013','TEST','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','Language',0,1,'ALL',0);

INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_Calc0','None',0,0,'Calculator feature is disabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',-1,'TDSCore_Staging_Configs_2013','TDSCore_Staging_Configs_2013','TEST','ALL',NULL);
INSERT INTO `client_testtool` (`clientname`,`type`,`code`,`value`,`isdefault`,`allowcombine`,`valuedescription`,`context`,`sortorder`,`origin`,`source`,`contexttype`,`testmode`,`equivalentclientcode`)
VALUES ('<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>','Calculator','TDS_CalcSciInv&TDS_CalcGraphingInv&TDS_CalcRegress','ScientificInv&GraphingInv&Regressions',1,0,'Regressions Calculator is enabled','<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',0,'TDSCore_Staging_Configs_2013','TDSCore_Staging_Configs_2013','TEST','ALL',NULL);

INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ESN',1,'Calculator','TDS_CalcSciInv&TDS_CalcGraphingInv&TDS_CalcRegress','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',1,'Calculator','TDS_CalcSciInv&TDS_CalcGraphingInv&TDS_CalcRegress','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ENU',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
INSERT INTO `client_tooldependencies` (`context`,`contexttype`,`iftype`,`ifvalue`,`isdefault`,`thentype`,`thenvalue`,`clientname`,`_key`,`testmode`)
VALUES ('<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>','TEST','Language','ESN',0,'Calculator','TDS_Calc0','<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',UNHEX(REPLACE(UUID(), '-', ''),'ALL');
</code>
</pre>
</div>

[back to TDS Configuration Tasks](index.html)