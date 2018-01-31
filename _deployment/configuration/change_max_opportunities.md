---
title: How to Change the Maximum Number of Opportunities for an Assessment
permalink: "deployment/configuration/change_max_opportunities.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Overview
The number of times a student can take an exam is configurable on a per-exam basis. That is, each exam can have a different number of maximum opportunities.

### Change the Maximum Number of Opportunities For a Single Assessment
To change the maximum number of times a student can take an exam for a specific assessment:

* run the following SQL:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
SET @assessmentLabel = '[<span class="placeholder">the display label (from the Assessment Selection screen) that you want to change</span>]';
SET @clientName = '[<span class="placeholder">the client that "owns" the assessment; <strong>SBAC</strong> for production; <strong>SBAC_PT</strong> for others</span>]';
SET @desiredMaxOpportunities = [<span class="placeholder">the maximum number of times a student can take the exam</span>];

START TRANSACTION;
UPDATE
    configs.client_testproperties
SET
    maxopportunities = @desiredMaxOpportunities
WHERE
    clientname = @clientName
    AND label = @assessmentLabel;
COMMIT;
</code>
</pre>
</div>

* Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
SET @assessmentLabel = '<span class="placeholder-example">G11 ELA Practice Test</span>';
SET @clientName = '<span class="placeholder-example">SBAC</span>';
SET @desiredMaxOpportunities = <span class="placeholder-example">1</span>;

START TRANSACTION;
UPDATE
    configs.client_testproperties
SET
    maxopportunities = @desiredMaxOpportunities
WHERE
    clientname = @clientName
    AND label = @assessmentLabel;
COMMIT;
</code>
</pre>
</div>


* Flush the Redis cache
* Restart the Student WAR pods

[back to TDS Configuration Tasks](index.html)