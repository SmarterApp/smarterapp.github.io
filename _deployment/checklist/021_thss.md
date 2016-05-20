---
title: Teacher Hand Scoring System (THSS) Installation Checklist
permalink: "/deployment/checklist/thss.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide interface for teachers to hand-score assessment items |
| Communicates With | OpenAM, TIS |
| Repository Location | [https://bitbucket.org/sbacoss/testintegrationsystem_release](https://bitbucket.org/sbacoss/testintegrationsystem_release) |
| Additional Documentation | [README](https://bitbucket.org/fwsbac/teacherhandscoresys_release), [reportxml_os.xsd](https://bitbucket.org/fwsbac/teacherhandscoresys_release/src/9208d85c5fc1281ff2bb11ba7bad7e07df5fcf23/Docs/reportxml_os.xsd?at=default&fileviewer=file-view-default), [SAML Templates](https://bitbucket.org/fwsbac/teacherhandscoresys_release/src/9208d85c5fc1281ff2bb11ba7bad7e07df5fcf23/Docs/SAML/?at=default) |

***NOTE:***{: style="color: #f00"} *The THSS database can reside on the same MSSQL Server that hosts the TIS databases.*{: style="color: #04384e"}

## Instructions

### Create the THSS Database
* Create the following database on the MSSQL Server:
  * `TSS`

#### Create THSS Databse User Account(s)
* Create a user account (i.e. an MSSQL login or an Active Directory user) that has access to read/write (`db_reader` and `db_writer` roles) for the database cited above.

#### Create the TSS Database Schema
* Run the following scripts in order against the `TSS` database:
  1. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Items.sql`
  2. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Dimensions.sql`
  3. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.ConditionCodes.sql`
  4. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Districts.sql`
  5. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Schools.sql`
  6. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Students.sql`
  7. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Teachers.sql`
  8. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Tests.sql`
  9. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Responses.sql`
  10. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Assignments.sql`
  11. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Logs.sql`
  12. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo._dbLatency.sql`
  13. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.fn_SplitDelimitedString.sql`
  14. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\*.sql`

### Create AWS Instance
* Create server instance to host Teacher Hand Scoring System components
  * AWS instance must be at least a **t2.small**
  * Select an image with the **Windows Server 2012 R2 Base**{: style="color: #04384e"} 64-bit operating system
* Create an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 1433
  * 80
  * 3389
  * 443

### Configure Windows AWS Instance
{% include checklist/windows_server_config.md %}

## Deploy the Teacher Hand Scoring System

### Build the Test Hand Scoring System
* Clone the THSS repository to a machine that can build .NET applications:
  * `hg clone https://`[*your BitBucket user or team name*{: style="color: #f00;"}]`/sbacoss/testintegrationsystem_release`
  * Example:
    * `hg clone https://`<span class="placeholder-example">jjohnson-fw@bitbucket.org</span>`/sbacoss/testintegrationsystem_release`
* Launch Visual Studio
* Open the `Src/TSS.sln`

{% include checklist/windows_vs_publish.md %}

### Deploy the THSS Component
* Connect to the AWS Windows server via RDP
* Create a directory where the THSS component will be deployed
  * The THSS Component is a web application, thus must be accessible by IIS
  * Example:  `C:\inetpub\`<span class="placeholder-example">thss</span>
* From the machine where the THSS web application was built, copy the *entire contents* of the directory where the **Publish** wizard deployed the component into the directory created in the previous step.

### Configure a TDS Receiver Website in IIS
* Launch the **Internet Information Services Manager**
  * Click **Start** button in lower-left corner (leftmost control on toolbar)
  * Click the down arrow (if the **Internet Information Services Manager** does not appear)
  * Search for and launch the **Internet Information Services Manager**
* Right click on the **Default Web Site**
  * Choose **Manage Website...**
  * Choose **Stop**
* Right click on **Sites** and choose **Add Website...**

![Add website within IIS](/res/images/checklist/iis_01_add_site.png){: width="100%"}

* Provide the following details in the dialog that appears:
  * Site name: [*Choose a meaningful name*{: style="color: #f00;"}]
  * Application Pool: [*Choose an exisitng Application Pool, or allow IIS to create a new Application Pool for this site (**recommended**)*{: style="color: #f00;"}]
  * Physical Path: [*The path to where the THSS web application components have been deployed*{: style="color: #f00;"}]
  * Binding:  [*Choose an available port*{: style="color: #f00;"}]
  * Host name:  [***OPTIONAL:*** *Provide a host name*{: style="color: #f00;"}]

* Screenshot showing example values for the TDS Receiver website:

![Add website details](/res/images/checklist/iis_07_thss_site_details.png){: width="100%"}

### Configure the TDS Receiver Application