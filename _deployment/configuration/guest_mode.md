---
title: How to Enable/Disable Guest Mode
permalink: "deployment/configuration/guest_mode.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Disabling Guest Mode/Practice Test
By default, guest login is enabled for the **SBAC_PT** client, and disabled for the **SBAC** client. This configuration flag
can be updated in the `configs.client_systemflags` table. For example, the following steps can be taken to enable guest login
for the **SBAC** client:

1. Update the `ison` column to `1` for the "AnonymousTestee" `auditobject`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UPDATE configs.client_systemflags
SET ison = 1
WHERE auditobject = 'AnonymousTestee'
AND clientname = '<span class="placeholder">[SBAC for production assessments, SBAC_PT for practice assessments]</span>';
</code>
</pre>
</div>

2. Flush the redis cache

[back to TDS Configuration Tasks](index.html)