---
title: Using the Smarter Balanced Secure FTP Site
dateCreated: 2015-01-09
date: 2015-04-20
version: 1.1
layout: document
author: Brandt Redd
---
# Introduction

Smarter Balanced uses a Secure FTP site provided by HostedFTP to share sensitive and bulky digital materials including Interim and Summative test packages. These instructions will help you set up your FTP client for access to these materials.

Most contents on the Secure FTP site are sensitive. For member states, these contents are subject to the confidentiality clause of your Memo of Understanding with UCLA/Smarter Balanced. For service providers and other partners, you should have signed a confidentiality agreement before being granted access.

## Welcome Email
The contents on the Secure FTP site are divided into folders and access is granted on a per-folder basis. Each time you are granted access you will receive a welcome email similar to the following:

    From: FTP-Admin [mailto:ftp@hostedftp.com] 
    Sent: Wednesday, January 7, 2015 1:12 AM
    To: Zog.Jones@farside.org
    Cc: FTP-Admin
    Subject: AssessmentTrainingAndOperations
    
    https://smarterbalanced.hostedftp.com/Xg48gdYRf2
    ~ click to login ~
    
    FTP-Admin (sbac) gave you Read access to the folder 
    AssessmentTrainingAndOperations
    
    Smarter Balanced Assessment Consortium
    http://www.smarterbalanced.org
    
    Hosted~FTP~
    FTP in the Cloud
    www.hostedftp.com

## Setting a Password
The first time you click on one of these links, you will be prompted to set a password. For future access, your username is your email address and your password is the one you set the first time you connect.

Per the terms of your organization's confidentiality agreement, you share with UCLA/Smarter Balanced the responsibility for preserving the security of these materials. So, please set a good quality password and keep it secure.

## Web Browser Access

**[ftps.smarterbalanced.org](http://ftps.smarterbalanced.org)**

You can access the Secure FTP site in your web browser either by clicking on an emailed link or by browsing to [ftps.smarterbalanced.org](http://ftps.smarterbalanced.org). Once on the site, click on the **Files** tab to find files and on the **Setup** tab to change your password or make other changes to your account.

From the **Files** tab you can download individual files or an entire folder. When downloading a folder, the site combines all of the files in the folder into a .zip file. 

## Configuring an FTP Client

An FTP client is the better choice for reliability and performance when downloading many files or bulky files. Any FTP client that supports encryption will work. [FileZilla](https://filezilla-project.org/), a free download, is an excellent choice.

Once you have installed an FTP client, you will need to configure the Smarter Balanced site. Here are the parameters:

    Host (or server) Name:  ftps.smarterbalanced.org
    Protocol:               FTP (File Transfer Protocol)
    Encryption:             Explicit FTP over TLS
    Logon Type:             Normal
    User:                   <your email address>
    Password:               <your password, set using the website>
    Default Directory:      /

## Other Tips
* Only the folders to which you have been granted access will appear regardless of whether you use a browser or an FTP client.

* You may be granted read-only or read-write access depending on your relationship with Smarter Balanced. This can change on a per-folder basis. So, you may have read access to one folder and read-write access to another.

* For security reasons, access is granted on an as-needed basis. If your need to access content changes, please notify Smarter Balanced personnel.

