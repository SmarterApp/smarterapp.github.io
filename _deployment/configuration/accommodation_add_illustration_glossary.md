---
title: How to Add an Illustration Glossary Accommodation
permalink: "deployment/configuration/accommodation_add_illustration_glossary.html"
layout: "document"
categories: ["student", "accommodations"]
---

## Overview
The Illustration Glossary provides the student with an image/drawing that describes a particular word or phrase.  In some cases, the illustration will have an accompanying definition.  The Illustration Glossary is displayed as a modal window that can have one or two tabs (one for the image, another for the definition).  The image and definition are contained within the item's content.

## Creating an Illustration Glossary Accommodation

### Illustration Glossary Creation Steps
To create a calculator accommodation for an assessment, take the following steps:

1.  Determine the `testid` of the assessment that should have a calculator assigned to it
3.  Copy the SQL for the desired calculator into an editor
4.  Replace the placeholders with appropriate values
5.  Execute the SQL against your database

### SQL For Creating an Illustration Glossary Accommodation
<div class="highlighter-rouge">
<pre class="highlight">
<code>
USE configs;

START TRANSACTION;
INSERT INTO client_testtooltype(
    clientname,
    toolname,
    allowchange,
    tideselectable,
    rtsfieldname,
    isrequired,
    tideselectablebysubject,
    isselectable,
    isvisible,
    studentcontrol,
    tooldescription,
    sortorder,
    dateentered,
    origin,
    source,
    contexttype,
    context,
    dependsontooltype,
    disableonguestsession,
    isfunctional,
    testmode,
    isentrycontrol)
VALUES (
    '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',
    'Illustration Glossary',
    1,
    1,
    'TDSAcc-ILG',
    0,
    0,
    1,
    1,
    0,
    NULL,
    0,
    NOW(),
    'configs_insert_ig_accommodation.sql',
    'configs_insert_ig_accommodation.sql',
    'TEST',
    '<span class="placeholder">[the value of the testid column from configs.tblsetofadminsubjects for the assessment]</span>',
    NULL,
    0,
    1,
    'ALL',
    0);
COMMIT;

START TRANSACTION;
INSERT INTO client_testtool(
    clientname,
    type,
    code,
    value,
    isdefault,
    allowcombine,
    valuedescription,
    context,
    sortorder,
    origin,
    source,
    contexttype,
    testmode,
    equivalentclientcode)
VALUES (
    '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',
    'Illustration Glossary',
    'TDS_ILG0',
    'Do not show Illustration Glossary',
    1,
    0,
    'Do not show Illustration Glossary',
    '[the value of the testid column from configs.tblsetofadminsubjects]',
    -1,
    'configs_insert_ig_accommodation.sql',
    'configs_insert_ig_accommodation.sql',
    'TEST',
    'ALL',
    NULL),
(
    '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>',
    'Illustration Glossary',
    'TDS_ILG1',
    'Show Illustration Glossary',
    0,
    0,
    'Show Illustration Glossary',
    '[the value of the testid column from configs.tblsetofadminsubjects]',
    1,
    'configs_insert_ig_accommodation.sql',
    'configs_insert_ig_accommodation.sql',
    'TEST',
    'ALL',
    NULL);
COMMIT;

START TRANSACTION;
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
    'testshell.aspx',
    'ClientSide',
    IFNULL(MAX(messageid), 0) + 1,
    'Student',
    'TDS.WordList.illustration',
    'Illustration',
    'AIR'
FROM
    tds_coremessageobject;
COMMIT;

START TRANSACTION;
INSERT INTO tds_coremessageuser(
    _fk_coremessageobject,
    systemid)
SELECT
    MAX(_key),
    'Student'
FROM
    tds_coremessageobject;
COMMIT;

-- needed in order to reset/recalc all the messages
START TRANSACTION;
DELETE FROM __appmessages;
DELETE FROM __appmessagecontexts;
COMMIT;
</code>
</pre>
</div>

[back to TDS Configuration Tasks](index.html)