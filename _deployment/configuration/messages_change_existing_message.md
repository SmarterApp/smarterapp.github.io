---
title: How to Update Text of an Existing Message
permalink: "deployment/configuration/messages_change_existing_message.html"
layout: "document"
categories: ["student", "tds-config"]
---

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

[back to TDS Configuration Tasks](index.html)