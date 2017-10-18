---
title:  TDS Deployment Checklist
permalink: "/deployment/checklist/student_3x/index.html"
categories: ["deployment", "checklist_index"]
---

# Overview

## Intended Audience
This document is intended for use by system administrators and/or software developers that want to deploy an instance of the Test Delivery System (TDS) to a set of server instances hosted on Amazon Web Services (AWS).

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
| Exam Results Transmitter &nbsp;| [3.1.4.RELEASE](https://github.com/SmarterApp/TDS_ExamResultsTransmitter/releases/tag/3.1.4.RELEASE) |
| Equation Scorer Service &nbsp;| [3.1.1.RELEASE](https://github.com/SmarterApp/TDS_ItemScoring/releases/tag/3.1.1.RELEASE) |  
| Configuration Service &nbsp;| [3.1.1.RELEASE](https://github.com/SmarterApp/SS_ConfigurationService/releases/tag/3.1.1.RELEASE) |  

# Deployment Checklists

## Shared Services
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "shared_services" and p.categories contains "3x" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

## Test Delivery System
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "tds" and p.categories contains "3x" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>