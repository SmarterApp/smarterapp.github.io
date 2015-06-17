---
title: Secure Browser Session Launch Protocol
dateCreated: 2015-06-11
date: 2015-06-11
version: 0.5
layout: document
author: Brandt Redd
---
### Document Status
This is a draft specification and subject to revision. Document history and previous versions may be found on the [SmarterApp GitHub Repository](https://github.com/SmarterApp). Before it is authoritative, the following steps must be completed:

|Step|Completion Date|
|----|---------------|
|Review by Smarter Balanced Member Representatives||
|Review by Secure Browser Contractor||
|Review by Test Delivery System Contractor||
|Implementation in Test Delivery System and Secure Browsers and Integration Testing Completed|

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in IETF RFC 2119.

# Introduction

This is a specification for launching a secure browser and initiating a session with the appropriate test delivery server URL without requiring pre-configuration of the secure browser. It is an enhancement to existing SmarterApp secure browsers and associated protocols.

## Background

The SmarterApp Secure Browsers are part of the SmarterApp open source assessment software suite maintained at [SmarterApp.org](http://www.smarterapp.org) sponsored by the [Smarter Balanced Assessment Consortium](http://www.smarterbalanced.org). The need to secure a device during assessment activities is a common one and Smarter Balanced is working to ensure these browsers can be used in contexts beyond the Smarter Balanced assessments.

UCLA-Smarter Balanced currently supplies open source code to secure browsers for the following platforms. Links to the source code repositories are on [SmarterApp.org](http://www.smarterapp.org):

* Microsoft Windows: XP to 8.1
* Apple Mac OS X 10.5 to 10.10
* Apple iOS 6 to 8
* Chrome OS v31 and later
* Linux – Recent versions of Ubuntu and Red Hat Enterprise
* Select devices running Android OS 4.04 to 4.4

In addition to supplying browsers for these platforms, UCLA-Smarter Balanced also publishes a secure browser specification and approves third-party secure browsers targeted at other devices.

* [A list of currently approved secure browsers is published here](http://www.smarterbalanced.org/test-taking-devices-approved-secure-browsers).
* [Current requirements and specifications for secure browsers are published here](http://www.smarterapp.org/specs/SecureBrowserSpecification.html)

A Secure Browser must do the following:

1.	Browse to the designated test-taking URL and act as the web browser user agent for the test delivery servers.
2.	Exclude the student from accessing any unauthorized service or application on the device. Generally speaking this includes all applications and tools except for Assistive Technology applications.
3.	Provide a security API, callable from JavaScript, which allows the testing server to assess the security status of the devices.
4.	Provide a text-to-speech API, callable from JavaScript, wich allows the testing application to make use of text-to-speech facilities provided by the device operating system or by the browser.

## Issue Addressed by This Specification

The current secure browsers are hard-coded to browse to a particular URL when launched. Since each member state in the Smarter Balanced Assessment Consortium independently contracts with a test delivery provider they have different URLs and must customize the browser software for each state. For desktop browsers this is inconvenient. For browsers targeted at app stores such as iOS, ChromeOS, and Android its a bigger problem since app stores are reluctant to carry multiple versions of the same application.

In the 2014-2015 testing year the workaround was to deploy a browser landing page and require the student to select their state before redirecting to the test delivery server. This was inconvenient for students, prone to errors, and created a single point of failure for participating states. It's also incompatible with the goal of extending use beyond Smarter Balanced.

The protocol in this specification provides a convenient way for a student or test administrator to specify the test delivery server and associated test session independent of any pre-configuration or common service.

# User Interaction

Upon launch of the secure browser the user is prompted for a Host Name and a Session ID. If the secure browser has been used previously on the device then the host name SHOULD be pre-filled with the name used in the previous session but the name SHOULD remain editable.

Here is a mockup of the User Interface:

<div style="background-color: white; width: 25em; margin-left: auto; margin-right: auto; border: 1px solid black; border-radius: 8px; text-align: center; padding:1em; margin: 1em;" >
Host Name<br/>
<input type="text" value="smartertest.org"/><br/>
&nbsp;<br/>
Session ID<br/>
<input type="text" value=""/><br/>
&nbsp;<br/>
Student ID<br/>
<input type="text" value=""/><br/>
&nbsp;<br/>
<input type="button" value="Begin Session"/>
</div>

In a typical testing scenario, using the SmarterApp Test Delivery System, the test administrator begins a testing session on the administration console. The session has an ID. Once the session has been started then each student opens the secure browser and enters the Session ID and their individual Student ID. As each student connects, the test administrator receives a notification on their administration console and authorizes that student to proceed with the test.

This scenario was used by secure browsers are hard-coded to the testing URL. The first page they displayed prompted for the Session ID and Student ID. When using the Secure Browser Launch Protocol, it's the browser that prompts for that information. And the addition of a Host Name removes the need for a hard coded URL.

There are other uses of the secure browser, beyond testing, where a Session ID or Student ID may not be relevant. In those cases, the fields could be left blank. Likewise, there may be scenarios where a password is required instead of authorization by the Test Administrator as in the above scenario. When a password is required it would be entered on a subsequent webpage displayed after the browser has been launched.

# Secure Browser Session Launch Protocol

Once the secure browser has collected the Host Name, Session ID, and Student ID, it uses the Secure Browser Session Launch protocol to initiate a session with the content server. (The content server is usually a test delivery server.)

The secure browser initiates the session with an HTTP GET request.

## Request

Here's the URL of a sample request.

    https://smartertest.org/browsersessionlaunch?sessionid=S123-22&studentid=smith5343

Here's the same request in the form that the browser transmits to the server.

    GET //browsersessionlaunch?sessionid=S123-22&studentid=smith5343
    Host: smartertest.org
    User-Agent: Mozilla/5.0 Gecko/20110201 SmarterApp/7.5
	Accept-Language: en

The request MUST use https encryption. The host name from the launch prompt is used for the host name in the URL and request. The Session ID and Student ID are included in the URL query string as the "sessionid" and "studentid" parameters. Leading and trailing whitespace SHOULD be trimmed from the "sessionid" and "studentid" before sending.

This example uses the following values:

* Host Name: smartertest.org
* sessionid: S123-22
* studentid: smith5343

The request SHOULD include the [HTTP Accept-Language header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) and the [HTTP User-Agent header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.43) along with other headers typically included in a browser request.

The User-Agent header SHOULD be the same as the underlying browser technology (often Mozilla) with "SmarterApp/*version*" appended to the end to indicate that it is a Secure Browser that meets SmarterApp secure browser specifications. *More detail regarding the user agent string will be provided in a subsequent update to the [Secure Browser Requirements and Specifications](http://www.smarterapp.org/specs/SecureBrowserSpecification.html)*.

## Successful Response

The server validates the request -- that the Session ID and and the Student ID are valid and appropriate. If the request is valid, it SHOULD respond with an HTTP "303 See Other" redirection response. Certain web development platforms MAY use an HTTP "302 Found" redirect response but 303 is preferred. Browsers MUST respond properly to either 302 or 303.

**HTTP Response**

    HTTP/1.1 303 See Other
    Location: https://smartertest.org/sessionpage
    Pragma: sessionid="S123-22"

The Location header specifies the URL of the page that should be displayed to the user. The host name in the location URL need not be the same as the initial server and the path may be any value. The protocol (scheme) MUST be https.

The response MUST include a [Pragma Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.32) with "sessionid" set to the same session ID value that was included in the request query string. If the pragma header is not present or the session ID doesn't match then the secure browser SHOULD reject the response and return to the session information prompt screen.

The response MAY include [HTTP Cookies](http://tools.ietf.org/html/rfc2109#section-4.2.2). If the response includes cookies, the browser MUST accept the cookies for use in future requests according to industry standard rules for cookie management.

## Error Response

If the server detects an error, it SHOULD respond with an HTTP "400 Bad Request" response. If the content-type header is set to "text/plain" then the body of the response SHOULD be displayed in the secure browser along with a new prompt for the session information.

**HTTP Response**

    HTTP/1.1 400 Bad Request
	Content-Type: text/plain; charset=utf-8
    
    Unknown session ID. Please check the Session ID value and try again.

Upon receiving this response, the secure browser SHOULD display something like the following:

<div style="background-color: white; width: 20em; margin-left: auto; margin-right: auto; border: 1px solid black; border-radius: 8px; text-align: center; padding:1em; margin: 1em;" >
<div style="margin-left: auto; margin-right: auto; width:18em; text-color:red; text-align:left; border: 1px solid black; padding: 6px;">
Unknown Session ID. Please check the Session ID value and try again.
</div>
&nbsp;<br/>
Host Name<br/>
<input type="text" value="smartertest.org"/><br/>
&nbsp;<br/>
Session ID<br/>
<input type="text" value="S123-22"/><br/>
&nbsp;<br/>
Student ID<br/>
<input type="text" value="smith5343"/><br/>
&nbsp;<br/>
<input type="button" value="Begin Session"/>
</div>

In multilingual settings the server should check for an [HTTP Accept-Language" header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) and return the error message in the preferred language.

## Other Responses

Any other response, *including an HTTP "200 OK" response* MUST be considered an error and the browser should return to the session information prompt.

# Security Considerations

To preserve test and student security, the following issues SHOULD be taken into account when implementing a secure browser.

## Preventing Access to an Unauthorized Website
 
Students should be prevented from using a secure browser to access websites other than the one designated by the test administrator. Since the student enters the host name this cannot be entirely prevented. However, the following features of the protocol help minimize misuse.

As specified in the protocol, the response to the HTTP GET request MUST be "303 See Other" or "302 Found" and it MUST include "Pragma: sessionid" with the correct session ID. Any other response MUST be rejected by the secure browser. This prevents students from entering something like "www.youtube.com" into the host name and successfully connecting to an alternative website.

The testing server SHOULD validate the session ID and user ID before accepting a session. This will prevent students from accessing servers intended for other institutions or students.

Anyone with access to this specification could create an alternative website that is accepted by the secure browser. However, for that to be a problem, a web programmer would have to create the site and convince a student to enter alternate session information. The consequence of this would be that the student would not log into the test but, instead, would be accessing another website. The test administrator should be able to detect that by monitoring their test console and noticing that the student hasn't connected to the session. Good proctoring practice is for the test administrator to periodically circle the room and observe student activity. And a student that doesn't log into the test would not get credit for the test.

It is also common practice for school network administrators to block access to unauthorized websites.

## Network Security

All transmissions in the session launch protocol and any transation carrying assessment or student data SHOULD encrypted using [HTTP over TLS](http://tools.ietf.org/html/rfc2818) (also known as HTTPS).