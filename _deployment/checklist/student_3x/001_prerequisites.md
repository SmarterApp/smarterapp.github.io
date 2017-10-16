---
title: Deployment Pre-Requisites
permalink: "/deployment/checklist/student_3x/prereqs.html"
layout: "document"
categories: ["deployment", "checklist", "3x"]
---

# Domain and SSL Certificate
* Select domain name
* Register domain name
* Purchase wildcard SSL certificate from a certificate authority

# Source Control
* Create GitHub account
* ***OPTIONAL:***  Create [GitHub](https://GitHub.com/){: target="_blank"} team and grant access to users

### Common GIT Commands
In multiple parts of the deployment checklist will refer to specific tags of an application.  This will be particular important in the sections where you have to build an image or are looking for scripts.

After you've cloned a git repository you can do the following to check out a tag.

1. Run `git fetch` which will fetch all the active tags and branches for the repository.
2. Run `git tag` to list all the tags available
3. Checkout the tag by running `git checkout tags/<tagname>`.  For example, if you wanted to check out the 3.1.4.RELEASE tag you'd run `git checkout tags/3.1.4.RELEASE` 

# Amazon Web Services (AWS)
* Create AWS account for creating/provisioning server instances
* ***OPTIONAL:***  Install AWS command-line tools

# Microsoft .NET
* All Microsoft.NET applications depend on the Microsoft.NET 4.5 Framework.
* Must have the ability to build Microsoft.NET 4.5 applications (e.g. via [Visual Studio](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx) or [msbuild](https://msdn.microsoft.com/en-us/library/dd393574.aspx))
  * This requirement only applies to the [Test Integration System (TIS)](https://github.com/SmarterApp/TDS_TestIntegrationSystem){:target="_blank"} and the [Teacher Hand-Scoring System (THSS)](https://github.com/SmarterApp/TDS_TeacherHandScoringSystem){:target="_blank"}

[back to Deployment Checklists](index.html)