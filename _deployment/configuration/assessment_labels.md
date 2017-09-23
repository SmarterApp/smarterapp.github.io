---
title: How to Change Assessment Labels
permalink: "deployment/configuration/assessment_labels.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Changing Assessment Labels in TDS
Assessment labels are defined in the `configs.client_testproperties` table. The following steps can be taken
to update the label for a particular assessment id:

1. Update the `label` column in the `config.client_testproperties`

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UPDATE configs.client_testproperties
SET label = '<span class="placeholder">[desired assessment label]</span>'
WHERE testid = '<span class="placeholder">[assessment id]</span>';
</code>
</pre>
</div>

2. Flush the redis cache

[back to TDS Configuration Tasks](index.html)