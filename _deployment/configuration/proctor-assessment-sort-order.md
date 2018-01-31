---
title: How to Change Assessment Sort Order in Proctor
permalink: "deployment/configuration/proctor_assessment_sort_order.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Setting the Sort Order of Assessments in the Proctor Application
The order in which assessments appear in the Proctor application is stored in the `configs.client_testproperties` table.  The `sortorder` column in `client_testproperties` drives the order in which the assessments appear, and is `NULL` by default.  To change the order in which assessments appear in the Proctor application, update the `sortorder` column in `configs.client_testproperties` to put the assessments in the desired order.  Shown below is some example SQL that can be used as a basis for conducting this update:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
START TRANSACTION;
UPDATE
    configs.client_testproperties
SET
    sortorder = [<span class="placeholder">the position/sequence in which the assessment should appear</span>]
WHERE
    testid = '[<span class="placeholder">the testid of the assessment to affect</span>]'
    AND clientname = '[<span class="placeholder">the client that "owns" the assessment; typically <strong>SBAC</strong> for production and <strong>SBAC_PT</strong> for others</span>]';
COMMIT;
</code>
</pre>
</div>

Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
START TRANSACTION;
UPDATE
    configs.client_testproperties
SET
    sortorder = <span class="placeholder-example">2</span>
WHERE
    testid = '<span class="placeholder-example">SBAC-IRP-CAT-ELA-3</span>'
    AND clientname = '<span class="placeholder-example">SBAC_PT</span>';
COMMIT;
</code>
</pre>
</div>

[back to TDS Configuration Tasks](index.html)