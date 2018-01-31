---
title: How to require the use of the Secure Browser v10
permalink: "deployment/configuration/require_secure_browser.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Instructions for Requiring the use of Secure Browser v10
In the configs database schema add a row to the client_systemflags table.

Add secure browser required setting
```sql
INSERT INTO configs.client_systemflags (auditobject, ison, description, clientname, ispracticetest, datechanged, datepublished) VALUES ('secureBrowserRequired', 1, 'Requires the use of secure browser to log in', 'SBAC_PT', true, now(), null);
```
To disable the secureBrowserRequired flag:
```sql
UPDATE configs.client_systemflags SET ison = false AND datechanged = now() WHERE auditobject = 'secureBrowserRequired' AND clientname = 'SBAC_PT'
```

[back to TDS Configuration Tasks](index.html)s