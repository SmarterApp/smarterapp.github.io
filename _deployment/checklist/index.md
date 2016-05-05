---
title:  TDS Deployment Checklist
permalink: "/deployment/checklist/index.html"
categories: ["deployment"]
layout: post
---

# Overview
TODO:  Write overview

## Intended Audience
This document is intended for use by system administrators and/or software developers that want to deploy an instance of the Test Delivery System (TDS) to a set of server instances hosted on Amazon Web Services (AWS).

## Conventions Used in the Checklists
The following conventions are used throughout the checklist:

* `Code` font indicates code or a command that should be executed on the command line.  For example, `sudo apt-get update` is an instruction to execute a command.
* [*some text*{: style="color: red"}] indicates text that should be replaced by the user.  When replacing text, the surrounding square brackets should be omitted.  For example, <span style="font-family: 'Lucida Console', Monaco, monospace">sudo hg clone ssh://hg@bitbucket.org/[*your bitbucket user or team name*{: style="color: red"}]/opendj_release</span> would look like `sudo hg clone ssh://hg@bitbucket.org/fwsbac/opendj_release` after the text has been replaced.
* ***OPTIONAL:***  These steps are not required to complete the deployment process.

## Assumptions
The following assumptions are made:

* The TDS will be deployed to server instances hosted on AWS
* The server instances will use the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system

# Deployment Checklists
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "checklist" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>