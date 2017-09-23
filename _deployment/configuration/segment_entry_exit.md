---
title: How to Chanage Assessment Segment Entry/Exit Approval
permalink: "deployment/configuration/segment_entry_exit.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Enabling Segment Entry/Exit Approval
Segment entry/exit approval is disabled by default for all segments. When enabled, a student will need to be approved by a
proctor before entering or exiting the segment. Segment entry/exit approval can be enabled for a particular segment by
taking the following actions:

1. Update the `entryapproval` or `exitapproval` flag for the particular segment id

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UPDATE configs.client_segmentproperties
SET entryapproval = 1
WHERE segmentid = '<span class="placeholder">[segment id]</span>';
</code>
</pre>
</div>

2. Flush the redis cache

[back to TDS Configuration Tasks](index.html)