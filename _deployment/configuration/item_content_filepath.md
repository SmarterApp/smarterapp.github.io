---
title: How to Update the Item Content File Path Casing
permalink: "deployment/configuration/item_content_filepath.html"
layout: "document"
categories: ["student", "tds-config"]
---

## Updating Item Content File Path Case
By default, the `itembank.loader_main()` stored procedure inserts items into the `itembank.tblitem` table with a lower-case "i"
file path directory. If there a mismatch in the item content path casing, the item file path can be updated for items by running the following
sql statement:

```
UPDATE itembank.tblitem
SET filepath = CONCAT("I", SUBSTRING(filepath, 2));
```

Executing the above statement will update all loaded items to use a capital "I" for the item content directory path.

[back to TDS Configuration Tasks](index.html)