---
title: How to Insert a New Message
permalink: "deployment/configuration/messages_insert_new_message.html"
layout: "document"
categories: ["student", "tds-config"]
---

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

[back to TDS Configuration Tasks](index.html)