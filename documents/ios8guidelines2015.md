---
title: Guidelines for the Configuration of iPads During Smarter Balanced Testing
dateCreated: 2015-02-24
date: 2015-02-24
version: 1.0
layout: document
author: AIR
---
For test security reasons, certain features of iOS devices need to be restricted administratively while students take Smarter Balanced assessments.  iOS 8.1.3 introduces configuration profile options you can use to restrict these features:

* Definition Lookup for Highlighted Words (“Dictionary”)
* Predictive Keyboard
* Spell-Check while Typing
* Auto-Correction while Typing

It is important to note, however, that in certain context while testing, similar functionality to these features is allowed (for example, in Writing assessments, students are allowed to check their spelling).  In these situations, the Smarter Balanced testing system offers customized features approved by Smarter and embedded in the system.  This ensures:
 
* Full compatibility across all platforms (i.e., spell-check should work exactly the same on all devices)
* Complete control of configuring which tests a feature would be available on (e.g., Smarter’s spell-check is only available for Writing)
* Approved content and functionality (e.g., embedded Merriam-Webster dictionary)

Here are the features’ profiles and recommended settings:


<table border="1" style="border: 1px solid black; border-collapse: collapse;">
<tr><td><b>Feature</b></td><td><b>Reason for Restriction</b></td><td><b>Profile Key</b></td><td><b>Recommended Value</b></td></tr>
<tr>
<td>Dictionary</td>
<td>This feature provides access to the Apple-provided dictionary, such as definitions for highlighted words. Instead, Smarter’s testing system offers the Merriam-Webster dictionary for approved tests.</td>
<td>&lt;key&gt;allowDefinitionLookup&lt;/key&gt;</td>
<td>False</td>
</tr>
<tr>
<td>Predictive Keyboard</td>
<td>iOS attempts to guess the word the user is likely to type next, based on its’ “learning” of the user’s writing style,  past conversations and current context, and offers the user a selection  of words.</td>
<td>&lt;key&gt;allowPredictiveKeyboard&lt;/key&gt;</td>
<td>False</td>
</tr>
<tr>
<td>Spell-Check</td>
<td>This feature will underline words iOS believes it misspelled.</td>
<td>&lt;key&gt;allowSpellCheck&lt;/key&gt;</td>
<td>False</td>
</tr>
<tr>
<td>Auto-Correction</td>
<td>This feature alerts the user with words iOS believes are incorrect.</td>
<td>&lt;key&gt;allowAutoCorrection&lt;/key&gt;</td>
<td>False</td>
</tr>
</table>

## Creating a Custom Profile

You can create a custom profile using a text editor (e.g., TextEdit) and install it on the iOS device that will be used for testing. If you are using a 3rd party MDM solution that allows custom profiles, you will need to consult your MDM vendor for instructions on how to import and install the profile. For more information about restricting these features, please refer to: [http://support.apple.com/en-us/HT204271](http://support.apple.com/en-us/HT204271).

You may also consult Apple's recommendations on the subject: [http://www.apple.com/education/docs/assessment_with_ipad.pdf](http://www.apple.com/education/docs/assessment_with_ipad.pdf)