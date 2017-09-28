---
title: Deleting the loader tables used to load test package
permalink: "deployment/configuration/delete_loader_data.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Clean up loader tables
The `itembank.loader_main()` stored procedure uses "loader" tables to load the XML data into a staging table environment prior to loading it into the actual `itembank` assesment tables.  Whenever the `itembank.loader_main()` fails to load a test package there is a chance that these tables will not be cleared which can make subsequent retries fail.  The SQL below will clear out those tables allowing one to retry loading the test package.

```
USE itembank;

DELETE from loader_setofitemstrands;
DELETE from loader_itemscoredimension;
DELETE from loader_itemscoredimensionproperties;
DELETE from loader_segment;
DELETE from loader_segmentblueprint;
DELETE from loader_segmentform;
DELETE from loader_segmentitemselectionproperties;
DELETE from loader_segmentpool;
DELETE from loader_segmentpoolgroupitem;
DELETE from loader_segmentpoolpassageref;
DELETE from loader_testblueprint;
DELETE from loader_testformgroupitems;
DELETE from loader_testformitemgroup;
DELETE from loader_testformpartition;
DELETE from loader_testformproperties;
DELETE from loader_testforms;
DELETE from loader_testitem;
DELETE from loader_testitemrefs;
DELETE from loader_testitempoolproperties;
DELETE from loader_testpackageproperties;
DELETE from loader_testpassages;
DELETE from loader_testpoolproperties;
DELETE from loader_testpackage;
```

[back to TDS Configuration Tasks](index.html)