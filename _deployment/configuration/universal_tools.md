---
title: How to Configure Universal Accommodations
permalink: "deployment/configuration/universal_tools.html"
layout: "document"
categories: ["student", "accommodations"]
---

## Configuring Universal Tools/Designated Supports/Accommodations for Specific Assessments and Segments
Accommodations, universal tools, and designated supports can be configured for specific segments or assessments. Test tool configurations
are specific to assessment/segment ids, meaning that new versions of the same assessment id should inherit the configuration of the
previous year. The three tables of concern are:

* `configs.client_testtooltype` - Defines a _category_ or "type" of tool (i.e., "American Sign Language")'
* `configs.client_testtool` - Defines the tool options based on [ISAAP codes](http://www.smarterapp.org/documents/ISAAP-AccessibilityFeatureCodes.pdf) (i.e., "TDS_ASL1" means "American Sign Language" is enabled)
    * For each tool "type", there are typically two or more rows in this table
    * If the `dependsontooltype` column is not null, that dependency will need to be defined in `configs.client_tooldependencies`
* `configs.client_tooldependencies` - Defines the set of dependency rules for a particular _type_ of tool, if such rules exist
    * The tooldependency rules are represented by the following conditional expression:
        * IF <`iftype`> IS <`ifvalue`> THEN <`thentype`> IS <`thenvalue`>
    * For example, IF <`Language`> IS <`ENU-Braille`> THEN <`Masking`> IS <`TDS_Masking0` (disabled)>
        * In other words, if "Braille" is selected as the language, then the masking tool should be disabled by default

The simplest way to configure accommodations/tools for a new assessment that does not have any tools configured is to select an existing
assessment that has the test tools defined that you wish to include, and copy them over to the new assessment. This can be
achieved by taking the following steps:

1. Identify the assessment id (known as the `testid` in the configuration database)
```
SELECT testid FROM configs.client_testproperties WHERE label = 'IRP CAT Grade 7 MATH';
>> SBAC-IRP-Mathematics-7
```
2. Copy the test tool data for the testid we identified
```
INSERT INTO configs.client_testtooltype
SELECT * FROM configs.client_testtooltype
WHERE context = 'SBAC-IRP-Mathematics-7';
```
```
INSERT INTO configs.client_testtool
SELECT * FROM configs.client_testtool
WHERE context = 'SBAC-IRP-Mathematics-7';
```
```
INSERT INTO configs.client_tooldependencies
SELECT * FROM configs.client_tooldependencies
WHERE context = 'SBAC-IRP-Mathematics-7';
```
3. Flush the redis cache

Example test tool seed data can be found in the [TDS_TestDeliverySystemDataAccess](https://raw.githubusercontent.com/SmarterApp/TDS_TestDeliverySystemDataAccess/develop/tds-dll-schemas/src/main/resources/import/genericsbacconfig/sbac_testtools.sql) github repository.

[back to TDS Configuration Tasks](index.html)