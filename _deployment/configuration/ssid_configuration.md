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

```
-- Change this value to whatever client you want to affect.
-- Typically this will be SBAC for Production environments or SBAC_PT for practice test environments
SET @clientName = 'SBAC';

START TRANSACTION;
UPDATE
    configs.client_testeeattribute
SET
    rtsname = 'ssid'
WHERE
    rtsname = 'ExternalID'
    AND clientname = @clientName;
COMMIT;
```

* Flush the Redis cache
* Restart the Student WAR application pods