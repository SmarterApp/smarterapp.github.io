---
title:  TDS Deployment Checklist
permalink: "/deployment/checklist/index.html"
categories: ["deployment", "checklist_index"]
date: 2017-10-07
---

# Overview

## Intended Audience
This document is intended for use by system administrators and/or software developers that want to deploy an instance of the Test Delivery System (TDS) to a set of server instances hosted on Amazon Web Services (AWS).

## Conventions Used in the Checklists
The following conventions are used throughout the checklist:

* `Code` font indicates code or a command that should be executed on the command line.<br>For example, `sudo apt-get update` is an instruction to execute a command.

* [*some text*{: style="color: red"}] indicates text that should be replaced by the user.  When replacing text, the surrounding square brackets should be omitted.<br>For example,

> <span style="font-family: 'Lucida Console', Monaco, monospace">sudo git clone ssh://git@github.com:[*your github user name*{: style="color: red"}]/IM_OpenDJ</span>
>
> would look like
>
> `sudo git clone ssh://git@github.com:`<span class="placeholder-example">hansolo</span>`/IM_OpenDJ`
>
> after the text has been replaced.
>
>

* <span class="placeholder-example">some text</span> indicates an example value for the [*some text*{: style="color: red"}] placeholder.
* ***IMPORTANT*:**{: style="color: #f00"} <span style=" background-color: #ff0;">These are example values for demonstration purposes only; the proper values for your environment may differ.</span>

* ***OPTIONAL:***  These steps are not required to complete the deployment process.

## Assumptions
The following assumptions are made:

* The Shared Services and Test Delivery System components will be deployed to server instances hosted on **AWS**{:style="color: #04384e"}
* The Linux server instances will use the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system
* The Windows server instances will use the **Microsoft Windows Server 2012 R2 Base**{: style="color: #04384e"} 64-bit operating system

## Disclaimer
The intent of these checklists are to install/deploy the Shared Services and Test Delivery System with minimal effort.  This document does not account for security considerations or network topology/layout of servers.  System administrators, network administrators, database administrators and/or developers involved in deploying the Shared Services and Test Delivery System are left to their own recognizance for identifying and following their own practices/standards for creating and securing their environment.

# Version Compatibility
TDS requires a suite of applications to be deployed.  The table below lists out the versions of the applications that work together.  References to building particular versions or deployment versions in the checklists refer to these versions unless specified.

| Project | Version | 
| :------- | :------- | 
| Student &nbsp; | [3.1.5.RELEASE](https://github.com/SmarterApp/TDS_Student/releases/tag/3.1.5.RELEASE) |
| Proctor &nbsp; | [3.1.4.RELEASE](https://github.com/SmarterApp/TDS_Proctor/releases/tag/3.1.4.RELEASE) |
| Assessment Service &nbsp;| [3.1.2.RELEASE](https://github.com/SmarterApp/TDS_AssessmentService/releases/tag/3.1.2.RELEASE) | 
| Session Service &nbsp; | [3.1.3.RELEASE](https://github.com/SmarterApp/TDS_SessionService/releases/tag/3.1.3.RELEASE) | 
| Config Service &nbsp; | [3.1.2.RELEASE](https://github.com/SmarterApp/TDS_ConfigService/releases/tag/3.1.2.RELEASE) | 
| Exam Service &nbsp;| [3.1.2.RELEASE](https://github.com/SmarterApp/TDS_ExamService/releases/tag/3.1.2.RELEASE) | 
| Student Service &nbsp;| [3.1.1.RELEASE](https://github.com/SmarterApp/TDS_StudentService/releases/tag/3.1.1.RELEASE) | 


# Deployment Checklists

## Shared Services
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "shared_services" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

## Test Delivery System
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "tds" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>