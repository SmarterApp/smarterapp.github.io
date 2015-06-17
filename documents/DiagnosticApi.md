---
title: SmarterApp Web Diagnostic API
dateCreated: 2015-06-02
date: 2015-06-02
version: 0.5
layout: document
author: Brandt Redd
---
### Document Status
This is a draft specification and subject to revision. Document history and previous versions may be found on the [SmarterApp GitHub Repository](https://github.com/SmarterApp). Before it is authoritative, the following steps must be completed:

|Step|Completion Date|
|----|---------------|
|Review by Smarter Balanced Member Representatives||
|Review by Test Delivery System Contractor||
|Implementation in at least one SmarterApp application|

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in IETF RFC 2119.

# Introduction

Web applications in the SmarterApp suite should have a diagnostic API that can be used to assure that installation is correct, that configuration is valid, that the system is operational, and that all communication channels are functioning properly. When the diagnostic fails, it should offer details that help systems administrators quickly diagnose and correct problems. The diagnostic API is also an appropriate endpoint for monitoring and alerting systems.

In order to be accessible under all circumstances, the diagnostic API is open; no authentication or authorization is required. Accordingly, the API must be designed in such a way to prevent disclosure of sensitive configuration information or its misuse for denial of service attacks.

## Document Conventions

This document uses the following conventions:

* *Italics* are used to represent placeholder values that are replaced with real values at runtime.
* *servicename.org* is a placeholder for the actual host or server name.
* The basic diagnostic API does not make use of XML namespaces. Extensions may do so.

# Diagnostic API

The diagnostic API is called using an HTTP GET verb to a specific URL. When possible, the URL SHOULD be "http://*servicename.org*/status". The call includes optional query string parameters. For example "http://*servicename.org*/status?level=1" indicates a level 1 diagnostic.

The diagnostic API MUST return an XML document. Therefore, the **Content-Type** header MUST be set to "text/xml; charset=utf-8". The XML document SHOULD be beautified for easy reading by humans. This means that elements SHOULD start on separate lines and nested elements SHOULD be indented.

The **level** query string parameter indicates the level of diagnostic to perform. Following sections describe the tests that should be performed at each level and the expected responses. Diagnostic levels are additive; a level 3 diagnostic performs all tests from levels 0, 1, 2 and 3.

## Request Parameters

The following query string parameters may be used on the request. Implementations may add system-specific request parameters.


|Name|Example|Values|Default|Description|
|:---|:------|:-----|:------|:----------|
|level|3|Integer 0 to 5 or higher|0|The level of the diagnostic to perform. This document defines values from 0 to 5. Specific implementations may support fewer or more values. Since levels are additive, a number higher than those supported should result in the highest supported diagnostic level. For example, requesting a level 10 might result in a level 5 diagnostic -- being the highest level supported in a particular implementation.|
|nextlevels|3,3,2|Comma-separated list of integers.|0|For levels 5 and above, the server MAY pass through diagnostic requests to services on which it depends. when doing so, it SHOULD take the next value from from this list and pass it thorough as the diagnostic **level** for the next provider service. If the list has additional values (separated by commas), then it removes the first value and passes the remainder of the list as the **nextlevels** parameter to the provider service. If the list only has one value then it passes the value through verbatim.

## Response Elements

The following XML elements may be included in the response. Implementations will add system-specific response elements.

|Name|Description|
|:---|:----------|
|\<status\>|The root element of the XML response, \<status\> MUST include the **level**, **unit**, **statusRating**, **statusText**, and **time** attributes. Nested instances of the \<status\> element may be included, especially when embedding the response from a provider service on which this service depends.|
|\<system\>|The results of a system-level diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|\<configuration\>|The results of a configuration diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|\<database\>|The results of a database diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|\<providers\>|Surrounds the results of a provider diagnostics. MUST include the **statusRating**, and **statusText** attributes.|
|\<error\>|A human-readable error message that describes something that went wrong. In order to facilitate problem solving, the error message SHOULD describe the error with as much detail as possible without violating security considerations. The \<error\> element MAY include **statusRating**, **statusText**, and/or **time** attributes.|

## Response Attributes

The following XML attributes are included in the response. Implementations will add system-specific response elements.

|Name|Example|Description|
|:---|:------|:----------|
|**unit**|webserver|The unit for which status is being reported. Typical values might be "webserver", "appserver", "apiserver", "busnesslogic", "databaseserver", etc.|
|**statusRating:**|4|Rating from 0 to 4 indicating the status of the element; 0 indicates total failure; 4 indicates ideal operation.|
|**statusText**|Ideal|Text description corresponding to **statusRating**. This is provided for the convenience of a human reader. See below for values and corresponding descriptions.|
|**time**|2015-06-02T15:34:39.0907913Z|The time at which the status response is generated. MUST be in [RFC 3339 format](https://www.ietf.org/rfc/rfc3339.txt). MUST be UTC timezone.|

Values for **statusRating** and **statusText** are the following:

|statusRating|statusText|Description|
|-----------:|----------|-----------|
|4|Ideal|System is operating in the ideal state.|
|3|Recovering|System is recovering from a previous problem, for example, data replication may be underway. All operations are functioning properly.|
|2|Warning|A problem is imminent (e.g. running out of disk space) or the system is compensating for a failed, redundant component. All operations are functional but one or more issues should be addressed to prevent a failure.|
|1|Degraded|System is partially functional but some operations may not work.|
|0|Failed|System is not presently functional.|

Test results are nested and the result values of **statusLevel**, and **statusText** in parent elements are aggregated from the subelements. In most cases, the lowest value from the nested tests is selected. However, when lower levels include redundant systems the aggregated value MIGHT be "2 Warning" even when a nested status is "0 Failed".

Implementations will generally include custom elements and attributes that represent more detail specific to a particular service. 

# Level 0 Diagnostic: Ping

A level zero diagnostic is simply a "ping" to indicate that the server code is operational and communicating. The server should return a simple response and not perform any additional diagnostics.

**Request**

    http://servicename.org/status?level=0

**Success Response**

	<status unit="webserver" level="0" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
	</status>

**Failure Response**

	<status unit="webserver" level="0" statusRating="0"
		statusText="failed" time="2015-06-02T15:34:39.0907913Z">
		<error>Widget failure.</error>
	</status>

# Level 1 Diagnostic: Local System

A level one diagnostic tests the local system or server. Possible information and tests include application info, disk space, available memory, CPU loading, and so forth.

**Request**

    http://servicename.org/status?level=1

**Success Response**

	<status unit="webserver" level="1" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
	</status>

**Warning Response**

	<status unit="webserver" level="1" statusRating="4" statusText="ideal"
		time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="500"
					mbtotal="76447" statusRating="2"
					statusText="Warning">
					<warning>
						Less than 5% of disk space remaining on
						volume "/dev/sda2".
					</warning>
				</volume>
			</localdisk>
		</system>
	</status>

In these example the \<applicationInfo\>, \<identity\>, \<uptime\>, \<localdisk\>, and \<volume\> elements are examples that might be used in an actual implementation. Application developers are likely to choose different names and/or report different information about the server. Developers should focus on features of the server, operating system, and application that are critical to reliable operation.

# Level 2 Diagnostic: Configuration

A level two diagnostic tests whether required configuration is in place and configuration values are in range.

**Request**

    http://servicename.org/status?level=2

**Success Response**

	<status unit="webserver" level="2" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
	</status>

**Failure Response**

	<status unit="webserver" level="2" statusRating="0"
		statusText="failed" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="0" statusText="failed">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="5"/>
			<error>Value of samplename2 is out of range!</error>
		</configuration>
	</status>

In these examples the \<setting\> elements are examples that might be used in an actual implementation. Application developers may choose to use different names and/or report different information. Care should be taken not to disclose sensitive configuration information (see Security Considerations below).

# Level 3 Diagnostic: Read Data

A level three diagnostic tests whether the system can read from the database. If the server is directly connected to a database, it should perform a read operation on that database. If it's connected by way of an intermediary service or API, it would perform a read operation through that API or service.

**Request**

    http://servicename.org/status?level=3

**Success Response**

	<status unit="webserver" level="3" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="4" statusText="ideal">
			<dbread responsetime="0.025S" />
		</database>
	</status>

**Failure Response**

	<status unit="webserver" level="3" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="0" statusText="failed">
			<error>Database read timeout after 30 seconds.</error>
		</database>
	</status>

In these examples the \<dbread\> element is an example that might be used in an actual implementation. Application developers may choose to use different names and/or report different information.

# Level 4 Diagnostic: Write Data

A level four diagnostic tests whether the system can write data to the database and whether it can read the data back with fidelity. If the server is directly connected to a database, it SHOULD perform the write and read operations on that database. If it's connected by way of an intermediary service or API, it SHOULD perform a write and read through that API or service.

The write and read should be performed to an area of the database that exercises all of the communications and security provisions but does not affect the fidelity of actual data.

**Request**

    http://servicename.org/status?level=4

**Success Response**

	<status unit="webserver" level="4" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="4" statusText="ideal">
			<dbread responsetime="0.025S" />
			<dbwrite responsetime="0.0263" />
			<dbread responsetime="0.0204" />
		</database>
	</status>

**Failure Response**

	<status unit="webserver" level="4" statusRating="4"
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="0" statusText="failed">
			<dbread responsetime="0.025S" />
			<dbwrite responsetime="0.0263" />
			<dbread responsetime="0.0204" />
			<error>
				Data read from the database does not match the data
				written.
			</error>
		</database>
	</status>

In these examples the \<dbread\>, and \<dbwrite\> elements are examples that might be used in an actual implementation. Application developers may choose to use different names and/or report different information.

# Level 5 Diagnostic: Dependencies

A level 5 diagnostic tests all of the providers on which the service depends. In an ideal case, those providers also implement the diagnostic API in which case this diagnostic invokes the corresponding diagnostics on the provider services and embeds the results. In many cases, they use other APIs and so appropriate, benign operations should be invoked.

**Request**

    http://servicename.org/status?level=5&nextlevels=1,0

**Success Response**

	<status unit="webserver" level="5" statusRating="4" 
		statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="4" statusText="ideal">
			<dbread responsetime="0.025S" />
			<dbwrite responsetime="0.0263" />
			<dbread responsetime="0.0204" />
		</database>
		<providers statusRating="4" statusText="ideal">
			<status unit="businesslogic" level="1" statusRating="4"
				statusText="ideal" time="2015-06-02T15:34:39.0907913Z">
				<system statusRating="4" statusText="ideal">
					<applicationinfo serviceurl="logic4.servicename.org"
						version="2015.3.1.17178" />
					<identity machinename="lgc4" processid="5768" />
					<uptime servertime="2015-06-02T20:39:31.043343Z"
						runningsince="2015-05-21T20:48:54Z"
						runningfor="P11DT23H50M37S" />
					<localdisk statusRating="4" statusText="Ideal">
						<volume path="/dev/sda1" mbavailable="52113"
							mbtotal="76447" statusRating="4"
							statusText="Ideal"/>
						<volume path="/dev/sda2" mbavailable="63015"
							mbtotal="76447" statusRating="4"
							statusText="Ideal"/>
					</localdisk>
				</system>
			</status>
		</providers>
	</status>

**Failure Response**

	<status unit="webserver" level="0" statusRating="4"
		statusText="failed" time="2015-06-02T15:34:39.0907913Z">
		<system statusRating="4" statusText="ideal">
			<applicationinfo serviceurl="server1.servicename.org"
				version="2015.3.1.17178" />
			<identity machinename="svr1" processid="5768" />
			<uptime servertime="2015-06-02T20:39:31.043343Z"
				runningsince="2015-05-21T20:48:54Z"
				runningfor="P11DT23H50M37S" />
			<localdisk statusRating="4" statusText="Ideal">
				<volume path="/dev/sda1" mbavailable="52113"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
				<volume path="/dev/sda2" mbavailable="63015"
					mbtotal="76447" statusRating="4"
					statusText="Ideal"/>
			</localdisk>
		</system>
		<configuration statusRating="4" statusText="ideal">
			<setting name="samplename1" value="1"/>
			<setting name="samplename2" value="2"/>
		</configuration>
		<database statusRating="4" statusText="ideal">
			<dbread responsetime="0.025S" />
			<dbwrite responsetime="0.0263" />
			<dbread responsetime="0.0204" />
		</database>
		<providers statusRating="0" statusText="failed">
			<error>
				Timeout attempting to connect with provider
				"logic4.servicename.org"
			</error>
		</providers>
	</status>

# Security Considerations

Systems administrators and monitoring programs must be able to use the Diagnostic API to validate installation and configuration at all times regardless of whether authentication and authorization systems are operational. Indeed, this API is likely to be used to diagnose problems with authentication and authorization. Therefore, it SHOULD be an open API without sophisticated access restrictions.

Hiding the API by a using a path other than "/status" is an appealing option. However "security by obscurity" is inadequate protection. Once the name is discovered it can be shared broadly.

Since potentially anyone on the internet can call the diagnostic API there need to be protections against misuse. Here are key considerations and guidelines. Specific implementations may need additional features.

**No sensitive information should be reported.** Information about uptime, most configuration parameters, names of provider services and so forth are generally NOT sensitive. However, each should be considered in the context of the service. Security configurations including keys, key identifiers, and so forth MAY be sensitive. In those cases, the API SHOULD test for presence of and function of configuration data without reporting the actual data.

**Protect against denial of service misuse.** The higher diagnostic levels include operations that may be costly in terms of CPU, disk, and database utilization. These are important tests and they SHOULD be included in the API. However, a malicious user might call these diagnostics repeatedly in order to deliberately impair server performance. There are several things that can be done to limit misuse while keeping the API open:

* Ensure that the diagnostic calls have sufficiently modest demand that they can be called regularly without impairing operations.
* Have a minimum period between resource-demanding tests and report the previous results within that period. For example, the service might limit the level 4 (database write) diagnostic to once per minute (or five minutes). If another level 4 diagnostic is invoked within the period, the API reports the previous result -- including the time when the previous test was performed.
* Use semaphores to serialize diagnostic tests. Unlike the principal service, it's not necessary to support simultaneous diagnostic operations.

**Make the diagnostics as comprehensive as possible.** In addition to preventing misuse, security also involves detecting and resolving problems in a timely manner. The diagnostics should detect and report as many issues as possible. For example, a diagnostic might verify that the database schema is correct (all tables, indices, and stored procedures are in place). It might interact with the database and detect performance bottlenecks and report them as warnings. Or it might detect changes to use patterns that are indicative of attacks.

**Integrate the diagnostic API into an overall monitoring and alerting plan.** The diagnostic API should be part of an overall plan for monitoring the network, server hardware, operating system, and application software.
