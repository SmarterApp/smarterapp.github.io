---
title: Assessment Package Types in the Smarter Balanced Distributions
dateCreated: 2015-01-17
date: 2015-10-02
version: 1.1
layout: document
author: Brandt Redd with input from David Lopez de Quintana
---
There are five types of assessment packages distributed by Smarter Balanced. These packages work in conjunction to inform the test delivery, scoring and reporting systems.

## Test Packages

* The **Content Package** is a package of the assessment items, stimulus passages, machine scoring rubrics, and associated metadata. It a [.zip file](https://en.wikipedia.org/wiki/Zip_%28file_format%29) constructed according to the [SmarterApp Item Packaging Specification](http://www.smarterapp.org/specs/SmarterApp_ItemPackaging.html) which is a profile of [IMS Content Packaging](http://www.imsglobal.org/content/packaging/). One content package may include items used by multiple tests.  

* The **Registration Package** is intended for loading into the a student registration system. It provides summary information about the test so that registration systems can list the test as available and accept student registrations or eligibility.

* The **Administration Package** is intended for loading into a test delivery system. It contains the list of assessment items that are to be included in the test and corresponding item metadata. The operational or embedded field test status of items is part of the metadata. The administration package also includes information about the order of item presentation. For fixed-form tests, this is a an ordered list of test segments with ordered items in each segment. For computer-adaptive tests, the administration package includes the test blueprint and all selection criteria required by the adaptive engine to select items for presentation.

* The **Scoring Package** is intended for loading into the test integration and test scoring system. Once all items are scored, the scoring engine uses the data from the scoring package to calculate test scores including the overall score, subscores for claims, and standard errors on each of these values. To facilitate this, the scoring package contains a list of all items in the test, much like the list in the administration package. The metadata indicates which items contribute to the score, and which should be ignored due to field test status or other reasons. 

* The **Reporting Package** contains information required to generate a test report. This includes the range of scores for the whole test and the range of scores that corresponds to each proficiency level.

Smarter Balanced summative tests and interim comprehensive tests have two parts - a performance task and a general test. For summative tests the general test is computer adaptive and is generally referred to as the "CAT" segment. For these tests there are multiple test administration packages. Usually there is a selection of available performance tasks per grade and there is one administration package for each performance task. Then there is another performance task for the general test. Since since one performance task and one general test are combined to generate a test score there is be one registration package, one scoring package, and one reporting package that correspond to the full set of administration packages for a particular grade and subject.

## Tabulator Tools

Smarter Balanced has produced open source tabulator tools that extract some of the data from these packages into tabular form for analysis and planning purposes.

* [TabulateSmarterTestContentPackage](https://github.com/SmarterApp/TabulateSmarterTestContentPackage/releases/latest) validates a test content package and produces a set of reports in .csv format suitable for loading into a spreadsheet or database. The reports include lists of assessment items, passages, and glossary terms including details about item types association with passages, and scoring methods. The metadata includes information about claim and target alignment but this should not be used for scoring purposes. The authoritative scoring data is in the scoring package.

* [TabulateSmarterTestAdminPackage](https://github.com/SmarterApp/TabulateSmarterTestAdminPackage/releases/latest) tabulates the items listed in a test administration package or test scoring package and produces reports in .csv format suitable for loading into a spreadsheet or database. This includes authoritative field test status for items, scoring method (hand or machine) and a wealth of other information about items. The tabulation reports do not include item order (for fixed form tests), selection criteria (for computer adaptive tests). For that information you must consult the test administration package directly.  