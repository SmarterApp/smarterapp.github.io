---
title: How to Change Student Login Screen Fields
permalink: "deployment/configuration/student_login_screen_fields.html"
layout: "document"
categories: ["student", "tds-config"]
---

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

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UPDATE configs.client_testeeattribute
SET atlogin = 'REQUIRE'
WHERE clientname = '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>'
AND label = 'Last Name';
</code>
</pre>
</div>

2. Flush the redis cache

[back to TDS Configuration Tasks](index.html)