---
title: Assessment Package Types in the Smarter Balanced Distributions
dateCreated: 2015-01-17
date: 2015-01-17
version: 1.0
layout: document
author: Brandt Redd mostly quoting David Lopez de Quintana
---
There are two main categories of assessment packages distributed by Smarter Balanced.

A **Test Package** is an XML file that represents a single test. It includes the test elements such as segments, test blueprints, segment blueprints, item pools, forms, etc. It does not include any item content, only references to items coupled with certain metadata needed to select items in the adaptive engine.

A **Content Package** is a package of the assessment items and associated metadata. It is constructed according to the [SmarterApp Item Packaging Specification](http://www.smarterapp.org/specs/SmarterApp_ItemPackaging.html) which is a profile of [IMS Content Packaging](http://www.imsglobal.org/content/packaging/).  

### Reduced Test Packages
There are several reductions of test packages for specific purposes. Each of these contains a subset of the information in the Test Package.

* **Registration Packages** are an abridged version of the test packages intended for loading into ART (or any vendor’s student registration system). They provides summary information about the test so that registration systems can add client-specific things like test windows, number of opportunities, and most importantly, student eligibility criteria. 

* **Administration Packages** are intended for loading into a test delivery system. They contain segments, test blueprints, segment blueprints, item and segment pools (ref only) and form/form partition declarations (ref only) that are required to administer a test to students. They do not contain test scoring rules because a scoring component is expected to scale score the tests.

* **Scoring Packages** contain all the scoring rules available for scoring a test.

* **Reporting Packages** contain information required to configure a reporting system.

