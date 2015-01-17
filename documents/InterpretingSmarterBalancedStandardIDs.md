---
title: Interpreting Smarter Balanced Standard IDs
dateCreated: 2015-01-17
date: 2015-01-17
version: 1.0
layout: document
author: Brandt Redd
---
References to the Smarter Balanced test blueprint are typically made by referencing the claim and the standard. However, in the item metadata references may also be in the form of "Standard IDs". Standard IDs include claim and target as well as additional information such as the corresponding [Common Core State Standard](http://www.corestandards.org). There are four forms of Standard IDs which are used for different purposes.

Here are two examples of standards references made in item metadata followed by instructions on how to interpret these values.

# Examples

## Math example:

    <StandardPublication>
        <Publication>SBAC-MA-v4</Publication>
        <PrimaryStandard>SBAC-MA-v4:1|NS|D-6|m|6.NS.6c</PrimaryStandard>
    </StandardPublication>
    <StandardPublication>
        <Publication>SBAC-MA-v6</Publication>
        <PrimaryStandard>SBAC-MA-v6:1|P|TS04|D-6</PrimaryStandard>
    </StandardPublication>

In this example, the standard alignment is represented in two formats. In each case, the format type is specified twice. Once in the &lt;Publication&gt; element and once as a prefix to the identifier with a colon separating the format type from the identifier.

## English Language Arts (ELA) example:

    <StandardPublication>
        <Publication>SBAC-ELA-v1</Publication>
        <PrimaryStandard>SBAC-ELA-v1:3-L|4-6|6.SL.2</PrimaryStandard>
    </StandardPublication>

# Interpreting the Standard Identifiers

----

## SBAC-MA-v4

* Use: Math primary standard identifier; used during authoring. Based on the content specifications hierarchy primary alignment to the standard level.
* Format: Claim&#124;Content Domain&#124;Target&#124;Emphasis&#124;Common Core Standard

In the example above this is interpreted as follows:

* Claim: 1
* Content Domain: NS
* Target: D-6
* Emphasis: m
* Common Core Standard: 6.NS.6c

The "-6" suffix on the target indicates 6th grade.

----

## SBAC-MA-v5

* Use: Math secondary standard identifier; used during authoring. Based on the content specifications hierarchy secondary alignment to the standard level.
* Format: Claim&#124;Content Domain&#124;Target&#124;Emphasis&#124;Common Core Standard

The format is the same as with v4. The only difference is this indicates a secondary, rather than a primary alignment.

----

## SBAC-MA-v6

* Use: Math standard identifier; used for delivery. Based on the blueprint hierarchy; does not go to standard level.
* Format: Claim&#124;Content Category&#124;Target Set&#124;Assessment Target

This format should be used for Math items in assessment delivery. It does not include the Common Core State Standard. Claim and Target should be the same as in the v4 version.

In the example above, this is interpreted as follows:

* Claim: 1
* Content Category: P
* Target Set: TS04
* Target: D-6

----

## SBAC-ELA-v1 ##

* Use: ELA; used for development and delivery.
* Format: Claim&#124;Target&#124;Common Core Standard

This is the only format for ELA items. It is used in both development and delivery.

In the example above, this is interpreted as follows:

* Claim: 3-L
* Target: 4-6
* Common Core Standard: 6.SL.2


As with Math, the "-6" suffix on the target indicates 6th grade.

Unlike Math, ELA claims include a suffix that indicates the domain. The possible claim/domain values are:

* 1-LT (Reading, Literary Text)
* 1-IT (Reading, Informational Text)
* 2-W (Writing)
* 3-L (Listen purposefully)
* 3-S (Speak purposefully)
* 4-CR (Conduct research)