---
title: How to Change Message Display Text
permalink: "deployment/configuration/messages_display_text.html"
layout: "document"
categories: ["student", "tds-config"]
---

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

[back to TDS Configuration Tasks](index.html)