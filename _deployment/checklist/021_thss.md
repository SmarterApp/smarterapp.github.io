---
title: Teacher Hand-Scoring System (THSS) Installation Checklist
permalink: "/deployment/checklist/thss.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide interface for teachers to hand-score assessment items |
| Communicates With | TIS |
| Repository Location | [https://bitbucket.org/sbacoss/testintegrationsystem_release](https://bitbucket.org/sbacoss/testintegrationsystem_release) |
| Additional Documentation | [TIS REST API](https://bitbucket.org/sbacoss/testintegrationsystem_release/src/a4efcfaad21c7dd154b8ab04bf452e0899c9aa13/TISServices/Documentation/TIS%20REST%20API.txt?at=default&fileviewer=file-view-default) |

## Instructions

### Create AWS Instance
* Create server instance to host Test Itegration Services (TIS) components
  * AWS instance must be at least a **t2.small**
  * Select an image with the **Windows Server 2012 R2 Base**{: style="color: #04384e"} 64-bit operating system
* Create an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 1433
  * 80
  * 3389
  * 443

### Configure Windows AWS Instance
{% include checklist/windows_server_config.md %}