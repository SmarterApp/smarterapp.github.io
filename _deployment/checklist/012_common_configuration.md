---
title: Common Configuration Steps
permalink: "deployment/checklist/commonconfiguration.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

# Common TDS Configuration Steps

The following are instructions for steps for common configuration changes in TDS.

## Changing Assessment Labels in TDS
Assessment labels are defined in the `configs.client_testproperties` table. The following steps can be taken
to update the label for a particular assessment id:

1. Update the `label` column in the `config.client_testproperties` 
```
UPDATE configs.client_testproperties 
SET label = '<desired assessment label>' 
WHERE testid = '<assessment id>'; 
``` 
2. Flush the redis cache


## Enabling Segment Entry/Exit Approval
Segment entry/exit approval is disabled by default for all segments. When enabled, a student will need to be approved by a 
proctor before entering or exiting the segment. Segment entry/exit approval can be enabled for a particular segment by 
taking the following actions:

1. Update the `entryapproval` or `exitapproval` flag for the particular segment id
```
UPDATE configs.client_segmentproperties 
SET entryapproval = 1 
WHERE segmentid = '<segment id>';
```
2. Flush the redis cache


## Configuring Universal Tools/Designated Supports/Accommodations for Specific Assessments and Segments
Accommodations, universal tools, and designated supports can be configured for specific segments or assessments. Test tool configurations
are specific to assessment/segment ids, meaning that new versions of the same assessment id should inherit the configuration of the 
previous year. The three tables of concern are:

* `configs.client_testtooltype` - Defines a _category_ or "type" of tool (i.e., "American Sign Language")'
* `configs.client_testtool` - Defines the tool options based on [ISAAP codes](http://www.smarterapp.org/documents/ISAAP-AccessibilityFeatureCodes.pdf) (i.e., "TDS_ASL1" means "American Sign Language" is enabled)
    * For each tool "type", there are typically two or more rows in this table
    * If the `dependsontooltype` column is not null, that dependency will need to be defined in `configs.client_tooldependencies`
* `configs.client_tooldependencies` - Defines the set of dependency rules for a particular _type_ of tool, if such rules exist
    * The tooldependency rules are represented by the following conditional expression:
        * IF <`iftype`> IS <`ifvalue`> THEN <`thentype`> IS <`thenvalue`>
    * For example, IF <`Language`> IS <`ENU-Braille`> THEN <`Masking`> IS <`TDS_Masking0` (disabled)> 
        * In other words, if "Braille" is selected as the language, then the masking tool should be disabled by default

The simplest way to configure accommodations/tools for a new assessment that does not have any tools configured is to select an existing 
assessment that has the test tools defined that you wish to include, and copy them over to the new assessment. This can be 
achieved by taking the following steps:

1. Identify the assessment id (known as the `testid` in the configuration database)
```
SELECT testid FROM configs.client_testproperties WHERE label = 'IRP CAT Grade 7 MATH';
>> SBAC-IRP-Mathematics-7
```
2. Copy the test tool data for the testid we identified
``` 
INSERT INTO configs.client_testtooltype 
SELECT * FROM configs.client_testtooltype
WHERE context = 'SBAC-IRP-Mathematics-7';
```
``` 
INSERT INTO configs.client_testtool
SELECT * FROM configs.client_testtool
WHERE context = 'SBAC-IRP-Mathematics-7';
```
```
INSERT INTO configs.client_tooldependencies
SELECT * FROM configs.client_tooldependencies
WHERE context = 'SBAC-IRP-Mathematics-7';
```
3. Flush the redis cache

## Disabling Guest Mode/Practice Test
By default, guest login is enabled for the **SBAC_PT** client, and disabled for the **SBAC** client. This configuration flag 
can be updated in the `configs.client_systemflags` table. For example, the following steps can be taken to enable guest login
for the **SBAC** client:

1. Update the `ison` column to `1` for the "AnonymousTestee" `auditobject`:
``` 
UPDATE configs.client_systemflags 
SET ison = 1
WHERE auditobject = 'AnonymousTestee'
AND clientname = 'SBAC';
```
2. Flush the redis cache

## Configuring Login Screen Input Fields
By default, the student application requires students to enter their (External) SSID and first name. The login screen is 
rendered based on the configuration settings defined in the `configs.client_testeeattribute` table. The login screen can 
be configured to require more (or less) student identifiers including:
- First name
- Last name
- Gender
- District
- Date of birth
- Grade
- SSID
- School
- State
- District

An example of how to require a student to enter/validate his or her's last name:
1. Update the `atlogin` column in `configs.client_testeeattribute` to **REQUIRE** 
``` 
UPDATE configs.client_testeeattribute
SET atlogin = 'REQUIRE'
WHERE clientname = 'SBAC'
AND label = 'Last Name';
```
2. Flush the redis cache

## Updating Item Content File Path Case
By default, the `itembank.loader_main()` stored procedure inserts items into the `itembank.tblitem` table with a lower-case "i" 
file path directory. If there a mismatch in the item content path casing, the item file path can be updated for items by running the following
sql statement:

``` 
UPDATE itembank.tblitem
SET filepath = CONCAT("I", SUBSTRING(filepath, 2));
```

Executing the above statement will update all loaded items to use a capital "I" for the item content directory path. 

## Updating Messages/Display Text
A number of configuration database tables are involved in the translation/message text system.
* `configs.tds_coremessageobject` - Primary message-related table containing the message key and default message (English)
    - The `_key` column contains the primary key **LONG** value, a foreign key to the `configs.client_messagetranslation` 
    and `configs.client_coremessageuser` table
    - The `appkey` column contains the message key string referenced in the application code
* `configs.client_messagetranslation` - Contains localized/translated messages 
    - keyed by the `_fk_coremessageobject` column, which is a foreign-key to `configs.tds_coremessageobject`. 
    - If there is a record for a message key present in this table, the message in `client_messagetranslation` will 
    override the default message defined in `configs.tds_coremessageobject`
* `configs.__appmessages` - A "cache" table where localized messages are copied into by the student and proctor 
    applications as the messages are encountered. 
    - This table should be cleared after making changes to the other message-related tables
* `configs.client_coremessageuser` - A table mapping a message key to a particular application
    - This table only needs to be inserted into when creating a new message
    
### Updating an Existing Message
Example: We want to update the message string "This is your score:" to "Here is your score:" 

1. Identify the message primary key for the message that needs to be updated
``` 
SELECT _key
FROM configs.tds_coremessageobject
WHERE message = 'This is your score:';
>>> 2521
```
2. Identify if there are any existing records in the translation table for the client and language
``` 
SELECT COUNT(*) 
FROM configs.client_messagetranslation
WHERE _fk_coremessageobject = 2421
AND clientname = 'SBAC_PT'
AND language = 'ENU';
>>> 0
``` 
3. Since there are no records, we can insert a record into the `configs.client_messagetranslation` table with our updated
message, which will override anything in `configs.tds_coremessageobject` for the current client and language.
``` 
INSERT INTO configs.client_messagetranslation (
    _fk_coremessageobject, 
    client, 
    message, 
    language, 
    grade, 
    subject, 
    _key, 
    datealtered)
VALUES (
    2421, 
    'SBAC_PT', 
    'Here is your score:', 
    'ENU', 
    '--ANY--', 
    '--ANY--', 
    SELECT UNHEX(REPLACE(UUID(), '-', '')), 
    NOW(3)
);
```
4. Clear the `configs.__appmessages` table
``` 
DELETE FROM configs.__appmessages;
```
5. Flush the redis cache


### Inserting a New Message
Example: We want to create a new label with the text "Illustration" with a ESN (spanish) translation of "Ilustración".

1. Insert a record into the `configs.tds_coremessageobject` table with our default "ENU" message:
``` 
INSERT INTO tds_coremessageobject(
    context,
    contexttype,
    messageid,
    ownerapp,
    appkey,
    message,
    fromclient
)
SELECT
    'testshell.aspx', -- context
    'ClientSide', -- contexttype
    IFNULL(MAX(messageid), 0) + 1, -- messageid
    'Student', -- ownerapp
    'TDS.WordList.illustration', -- appkey
    'Illustration', -- message
    'AIR' -- fromclient
FROM
    tds_coremessageobject;
```
2. Insert a record into the `configs.tds_coremessageuser` table, linking the key we generated in the SQL statement in step (1)
``` 
INSERT INTO tds_coremessageuser(
    _fk_coremessageobject,
    systemid)
SELECT
    MAX(_key),
    'Student'
FROM
    tds_coremessageobject;
```
3. Insert a record into the `configs.client_messagetranslation` table for the ESN translation
``` 
INSERT INTO client_messagetranslation(
    _fk_coremessageobject,
    client,
    message,
    language,
    grade,
    subject,
    _key,
    datealtered)
SELECT 
    MAX(_key), -- _fk_coremessageobject
    'SBAC_PT', -- client
    'Ilustración', -- message
    'ESN',     -- language 
    '--ANY--', -- grade
    '--ANY--', -- subject
    SELECT UNHEX(REPLACE(UUID(), '-', '')), -- _key, a random UUID
    NOW(3)     -- datealtered - the current datetime
FROM 
    tds_coremessageobject;
```
4. Clear the `configs.__appmessages` table
``` 
DELETE FROM configs.__appmessages;
```
5. Flush the redis cache

[back to Deployment Checklists](index.html)