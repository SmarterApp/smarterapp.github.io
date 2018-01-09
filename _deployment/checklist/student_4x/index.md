---
title:  TDS Deployment Checklist
permalink: "/deployment/checklist/student_4x/index.html"
categories: ["deployment", "checklist_index"]
---

# Overview

## Intended Audience
This document is intended for use by system administrators and/or software developers that want to deploy an instance of the Test Delivery System (TDS) to a set of server instances hosted on Amazon Web Services (AWS).

# Version Compatibility
TDS requires a suite of applications to be deployed.  The table below lists out the versions of the applications that work together.  References to building particular versions or deployment versions in the checklists refer to these versions unless specified.

Docker images are tagged with the same version as listed here.  The only project that does not have a docker image is the Proctor application.  This is due to the need to build the image with the security information to connect to OpenAM.  Those steps are covered in the Student and Proctor deployment checklist.

| Project | Version |
| :------- | :------- |
| Student &nbsp; | [4.0.6.RELEASE](https://github.com/SmarterApp/TDS_Student/releases/tag/4.0.6.RELEASE) |
| Proctor &nbsp; | [4.1.0.RELEASE](https://github.com/SmarterApp/TDS_Proctor/releases/tag/4.1.0.RELEASE) |
| Assessment Service &nbsp;| [4.0.0.RELEASE](https://github.com/SmarterApp/TDS_AssessmentService/releases/tag/4.0.0.RELEASE) |
| Session Service &nbsp; | [3.1.3.RELEASE](https://github.com/SmarterApp/TDS_SessionService/releases/tag/3.1.3.RELEASE) |
| Config Service &nbsp; | [4.1.0.RELEASE](https://github.com/SmarterApp/TDS_ConfigService/releases/tag/4.1.0.RELEASE) |
| Exam Service &nbsp;| [4.1.3.RELEASE](https://github.com/SmarterApp/TDS_ExamService/releases/tag/4.1.3.RELEASE) |
| Student Service &nbsp;| [4.0.3.RELEASE](https://github.com/SmarterApp/TDS_StudentService/releases/tag/4.0.3.RELEASE) |
| Content Service &nbsp;| [1.0.6.RELEASE](https://github.com/SmarterApp/TDS_ContentService/releases/tag/1.0.6.RELEASE) |
| Exam Results Transmitter &nbsp;| [4.1.1.RELEASE](https://github.com/SmarterApp/TDS_ExamResultsTransmitter/releases/tag/4.1.1.RELEASE) |
| Equation Scoring Service &nbsp;| [3.1.1.RELEASE](https://github.com/SmarterApp/TDS_ItemScoring/releases/tag/3.1.1.RELEASE) |
| Configuration Service &nbsp;| [3.1.1.RELEASE](https://github.com/SmarterApp/SS_ConfigurationService/releases/tag/3.1.1.RELEASE) |

# Deployment Checklists

## Shared Services
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "shared_services" and p.categories contains "4x" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

## Test Delivery System
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "tds" and p.categories contains "4x" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>