---
title: How to Change Student Login to Allow for SSID Instead of External SSID
permalink: "deployment/configuration/ssid_login_configuration.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Overview
When a new student is added to the system in the Administration and Registration Tools application (ART), they can configured with two identifiers:

* SSID
* External SSID

When a new system is deployed, the database is configured to require a student to log in with their External SSID.  In some cases, it may be preferable to have students sign into the student application with their SSID instead of their External SSID.  Configuration for which value should be used to sign into the student application is stored in the `configs.client_testeeattribute` table.

## Changing Configuration to Allow Students to Sign into Student With SSID Instead of External SSID
To allow a student to sign into the Student application with its SSID instead of the External SSID, take the following steps:

* Run the following SQL against the `configs` database:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
-- Change this value to whatever client you want to affect.
-- Typically this will be SBAC for Production environments or SBAC_PT for practice test environments
SET @clientName = '<span class="placeholder">[the client that "owns" the assessment; typically <strong>SBAC</strong> for production and <strong>SBAC_PT</strong> for others]</span>';

START TRANSACTION;
UPDATE
    configs.client_testeeattribute
SET
    rtsname = 'ssid'
WHERE
    rtsname = 'ExternalID'
    AND clientname = @clientName;
COMMIT;
</code>
</pre>
</div>

Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
-- Change this value to whatever client you want to affect.
-- Typically this will be SBAC for Production environments or SBAC_PT for practice test environments
SET @clientName = '<span class="placeholder-example">SBAC</span>';

START TRANSACTION;
UPDATE
    configs.client_testeeattribute
SET
    rtsname = 'ssid'
WHERE
    rtsname = 'ExternalID'
    AND clientname = @clientName;
COMMIT;
</code>
</pre>
</div>

* Flush the Redis cache
* Restart the Student WAR application pods

## Caveats
This configuration setting only affects which SSID the student logs in with.  The open-source TDS has been built on the assumption that the student's **External SSID** is what will be used as the student's primary identifier.  As such, the student's **External SSID** is primarily what's used to identify a student throughout the system.  Some examples of this include:

* When viewing students in the Proctor application, the student's **External SSID** will be displayed
* In the `exam` database (the database that supports the Exam service), the student's **External SSID** will be stored
* When the Exam Results Transmitter sends a Test Results Transmission (TRT) file to downstream systems, both identifiers (i.e the **SSID** and the **External SSID**) will be included

[back to TDS Configuration Tasks](index.html)