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


## Test Tool Configuration Stored Procedures
The following stored procedures in the `configs` database can be used to configure assessments with predefined sets of test tools.

Please note that the "client name" is also referred to as the "publisher" in the test specification package. The assessment 
id is also known as the "test id" in both the TRT and the test specification package. The "test id" value is shared between different 
years and publication of the same test/assessment. These values can be found in the `itembank.tblsetofadminsubjects` and `configs.client_testproperties` tables.

* `InsertGeneralTools('<client name>', '<assessment id>')`
   - Clears and then inserts universal tools and non-braille or subject specific accommodations/designated supports for the specified client and assessment.
   - This stored procedure should be executed before any of the following stored procedures
* `InsertBrailleTools('<client name>', '<assessment id>')`
   - Inserts braille tools and "ENU-Braille" language for the specified client and assessment
* `InsertSpanishTool('<client name>', '<assessment id>')`
   - Inserts the spanish ("ESN") language for the specified client and assessment
* `InsertCalculatorTool('<client name>', '<assessment/segment id>', <grade>, <isSegment>)` 
   - Inserts the calculator tool for the specified client, assessment (or segment), and grade. 
   - If the provided id is a segment id, the final argument should be `1`. 
   - The calculator type is dependent on the provided grade.
       * Grades 1 - 6: Basic Calculater
       * Grades 7 - 9: Scientific Calculator
       * Grades 10 - 12: Scientific/Inverse Regression calculator
* `InsertDictionaryTool('<client name>', '<assessment/segment id>', <grade>, <isSegment>)` 
   - Inserts the dictionary/thesaurus tools for the specified client, assessment (or segment), and grade. 
   - If the provided id is a segment id, the final argument should be `1`. 
   - The dictionary type is dependent on the provided grade.
       * Grades 1 - 6: Elementary Dictionary
       * Grades 7 - 9: Intermediate Dictionary
       * Grades 10 - 12: College Dictionary
   - The thesaurus tool is the same for all grade levels
* `ClearTools('<client name>', '<assessment id>')` 
   - Clears all tools except "Language" and "Print Size"
   
Below is an example of setting up tools for a performance math exam with calculator, spanish, and braille enabled.

``````
CALL configs.InsertGeneralTools('SBAC_PT', 'SBAC-Perf-MATH-11');
CALL configs.InsertBrailleTools('SBAC_PT', 'SBAC-Perf-MATH-11');
CALL configs.InsertSpanishTool('SBAC_PT', 'SBAC-Perf-MATH-11');
-- Calculator only included in the second segment
CALL configs.InsertCalculatorTool('SBAC_PT', 'SBAC-SEG2-MATH-11', 11, 1);
`````` 

**NOTE** - After making any test tool changes, be sure to flush the redis cache and restart student pods to clear all possible caching.

Example test tool seed data can be found in the [TDS_TestDeliverySystemDataAccess](https://raw.githubusercontent.com/SmarterApp/TDS_TestDeliverySystemDataAccess/develop/tds-dll-schemas/src/main/resources/import/genericsbacconfig/sbac_testtools.sql) github repository.

[back to TDS Configuration Tasks](index.html)