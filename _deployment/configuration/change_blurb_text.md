---
title: How to Change "Blurb" Text on the Student Login Screen
permalink: "deployment/configuration/change_blurb_text.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Overview
The "blurb" text is a section on the Student login screen displayed on the right-hand side of the login fields.

## Removing the "Blurb" Text From the Login Screen
To remove the "blurb" text from the Student login screen entirely, the message needs to be updated to empty, non-visible text.  A typical example is to change the message to `&nbsbp;`, a non-breaking space.  To suppress the "blurb" text, take the following steps:

1.  Copy the SQL below into an editor
2.  Update the following values:
* `@message`: Set this to `&nbsp;`
* `@languageCode`: Replace `[language code]` the language code representing the language of the message text (e.g. ENU, ESN, HAW)
* `@clientName`: Set this to **SBAC** for production environemnts or **SBAC_PT** for other environments
3.  Execute the SQL against your database
4.  Flush the redis cache

### SQL For Removing the "Blurb" Text
<div  class="highlighter-rouge">
<pre class="highlight">
<code>
USE configs;

SET @message = '&nbsp;';
SET @languageCode = '<span class="placeholder">[language code]</span>';
SET @clientName = '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>';

START TRANSACTION;
DELETE
    mt
FROM
    configs.client_messagetranslation mt
JOIN
    configs.tds_coremessageobject cmo
    ON mt._fk_coremessageobject = cmo._key
WHERE
    cmo.appkey = 'StaticContent.Label.PTIntroText'
    AND cmo.`contexttype` = 'ServerSide'
    AND mt.`language` = @languageCode;

INSERT configs.client_messagetranslation(_fk_coremessageobject, `client`, message, `language`, grade, `subject`, _key, datealtered)
SELECT
    cmo._key,
    @clientName,
    @message,
    @languageCode,
    '--ANY--',
    '--ANY--',
    UNHEX(REPLACE(UUID(), '-', '')),
    NOW()
FROM
    configs.tds_coremessageobject cmo
JOIN
    configs.client_messagetranslation mt
    ON (mt._fk_coremessageobject = cmo._key)
WHERE
    cmo.appkey = 'StaticContent.Label.PTIntroText'
    AND cmo.contexttype = 'ServerSide'
GROUP BY
    cmo._key;

UPDATE
    configs.tds_coremessageobject
SET
    message = @message,
    datealtered = NOW()
WHERE
    appkey = 'StaticContent.Label.PTIntroText'
    AND contextType = 'ServerSide';
COMMIT;

START TRANSACTION;
DELETE FROM __appmessages;
DELETE FROM __appmessagecontexts;
COMMIT;
</code>
</pre>
</div>

## Changing the "Blurb" Text on the Login Screen
For the "blurb" text and container to properly render on the Student loging screen, the desired message text must be in the following format:
<div  class="highlighter-rouge">
<pre class="highlight">
<code>
&lt;div class="blurb"&gt;<span class="placeholder">[Your message text goes here.  Can be any valid HTML (e.g. link, list, etc)]</span>&lt;/div&gt;
</code>
</pre>
</div>

Example:
<div  class="highlighter-rouge">
<pre class="highlight">
<code>
&lt;div class="blurb"&gt;<span class="placeholder-example">Click &lt;a href="http://link/to/somewhere"&gt;here&lt;/a&gt; if you are a California student.</span>&lt;/div&gt;
</code>
</pre>
</div>
To change the "blurb" text message in the Student login screen, take the following steps:

1.  Copy the SQL below into an editor
2.  Update the following values:
* `@message`: Replace `[Message text goes here.  Message text can be any HTML]` with the desired text
* `@languageCode`: Replace `[language code]` the language code representing the language of the message text (e.g. ENU, ESN, HAW)
* `@clientName`: Set this to **SBAC** for production environemnts or **SBAC_PT** for other environments
3.  Execute the SQL against your database
4.  Flush the redis cache

### SQL For Changing the "Blurb" Text
<div  class="highlighter-rouge">
<pre class="highlight">
<code>
USE configs;

SET @message = '&lt;div class="blurb"&gt;<span class="placeholder">[Message text goes here.  Message text can be any HTML]</span>&lt;/div&gt;';
SET @languageCode = '<span class="placeholder">[language code]</span>';
SET @clientName = '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>';

START TRANSACTION;
DELETE
    mt
FROM
    configs.client_messagetranslation mt
JOIN
    configs.tds_coremessageobject cmo
    ON mt._fk_coremessageobject = cmo._key
WHERE
    cmo.appkey = 'StaticContent.Label.PTIntroText'
    AND cmo.`contexttype` = 'ServerSide'
    AND mt.`language` = @languageCode;

INSERT configs.client_messagetranslation(_fk_coremessageobject, `client`, message, `language`, grade, `subject`, _key, datealtered)
SELECT
    cmo._key,
    @clientName,
    @message,
    @languageCode,
    '--ANY--',
    '--ANY--',
    UNHEX(REPLACE(UUID(), '-', '')),
    NOW()
FROM
    configs.tds_coremessageobject cmo
JOIN
    configs.client_messagetranslation mt
    ON (mt._fk_coremessageobject = cmo._key)
WHERE
    cmo.appkey = 'StaticContent.Label.PTIntroText'
    AND cmo.contexttype = 'ServerSide'
GROUP BY
    cmo._key;

UPDATE
    configs.tds_coremessageobject
SET
    message = @message,
    datealtered = NOW()
WHERE
    appkey = 'StaticContent.Label.PTIntroText'
    AND contextType = 'ServerSide';
COMMIT;

START TRANSACTION;
DELETE FROM __appmessages;
DELETE FROM __appmessagecontexts;
COMMIT;
</code>
</pre>
</div>

[back to TDS Configuration Tasks](index.html)