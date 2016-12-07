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

# Diagnostic API

The diagnostic API is called using an HTTP GET verb to a specific URL. When possible, the URL SHOULD be "http://*servicename.org*/status". The call includes optional query string parameters. For example "http://*servicename.org*/status?level=1" indicates a level 1 diagnostic.

The diagnostic API MUST return a JSON document. Therefore, the **Content-Type** header MUST be set to "application/json; charset=utf-8". The XML document SHOULD be beautified for easy reading by humans. This means that elements SHOULD start on separate lines and nested elements SHOULD be indented.

The **level** query string parameter indicates the level of diagnostic to perform. Following sections describe the tests that should be performed at each level and the expected responses. Diagnostic levels are additive; a level 3 diagnostic performs all tests from levels 0, 1, 2 and 3.

## Request Parameters

The following query string parameters may be used on the request. Implementations may add system-specific request parameters.


|Name|Example|Values|Default|Description|
|:---|:------|:-----|:------|:----------|
|level|3|Integer 0 to 5 or higher|0|The level of the diagnostic to perform. This document defines values from 0 to 5. Specific implementations may support fewer or more values. Since levels are additive, a number higher than those supported should result in the highest supported diagnostic level. For example, requesting a level 10 might result in a level 5 diagnostic -- being the highest level supported in a particular implementation.

## Response Properties

The following JSON properties may be included in the response. Implementations will add system-specific response elements.

|Name|Type|Example|Description|
|:---|:---|:------|:----------|
|**unit**|string|Student|The application or component for which status is being reported. Typical values might be "Student", "Proctor", "ART", etc.|
|**level**|integer|5|Diagnostic level used to produce the results.|
|**statusRating**|integer|4|Rating from 0 to 4 indicating the status of the element; 0 indicates total failure; 4 indicates ideal operation.|
|**statusText**|string|Ideal|Text description corresponding to **statusRating**. This is provided for the convenience of a human reader. See below for values and corresponding descriptions.|
|**dateTime**|string|2015-06-02T15:34:39.0907913Z|The time at which the status response is generated. MUST be in [RFC 3339 format](https://www.ietf.org/rfc/rfc3339.txt). MUST be UTC timezone.|
|**localSystem**|object||The results of a system-level diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|**configuration**|object||The results of a configuration diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|**database**|object||The results of a database diagnostic. MUST include the **statusRating**, and **statusText** attributes.|
|**providers**|object||Surrounds the results of a provider diagnostics. MUST include the **statusRating**, and **statusText** attributes.|
|**error**|object||A human-readable error message that describes something that went wrong. In order to facilitate problem solving, the error message SHOULD describe the error with as much detail as possible without violating security considerations. The **error** element MAY include **statusRating**, **statusText**, and/or **dateTime** values.|

## Response Values

The following values are included in the response. Implementations will add system-specific response elements.

Values for **statusRating** and **statusText** are the following:

|statusRating|statusText|Description|
|-----------:|----------|-----------|
|4|Ideal|System is operating in the ideal state.|
|3|Recovering|System is recovering from a previous problem, for example, data replication may be underway. All operations are functioning properly.|
|2|Warning|A problem is imminent (e.g. running out of disk space) or the system is compensating for a failed, redundant component. All operations are functional but one or more issues should be addressed to prevent a failure.|
|1|Degraded|System is partially functional but some operations may not work.|
|0|Failed|System is not presently functional.|

Test results are nested and the result values of **statusRating**, and **statusText** in parent elements are aggregated from the subelements. In most cases, the lowest value from the nested tests is selected. However, when lower levels include redundant systems the aggregated value MIGHT be "2 Warning" even when a nested status is "0 Failed".

Implementations will generally include custom elements and attributes that represent more detail specific to a particular service.

# Level 0 Diagnostic: Ping

A level zero diagnostic is simply a "ping" to indicate that the server code is operational and communicating. The server should return a simple response and not perform any additional diagnostics.  If no level value is provided the server SHOULD default to a Level 0 response.

**Request**

    http://servicename.org/status?level=0

**Success Response**

	{
		"unit": "Student",
		"level": 0,
		"dateTime": "Wed Nov 30 22:26:49 UTC 2016",
		"statusText": "Ideal",
		"statusRating": 4
	}

**Failure Response**

	{
		"unit": "Student",
		"level": 0,
		"dateTime": "Wed Nov 30 22:26:49 UTC 2016",
		"statusText": "Failed",
		"statusRating": 0
	}

# Level 1 Diagnostic: Local System

A level one diagnostic tests the local system or server. Possible information and tests include application info, disk space, available memory, CPU loading, and so forth.

**Request**

    http://servicename.org/status?level=1

**Success Response**

	{
		"unit": "Student",
		"level": 1,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Ideal",
		"statusRating": 4
	}

**Warning Response**

	{
		"unit": "Student",
		"level": 1,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Warning",
					"statusRating": 2,
					"warning": "Current free space is below the warning percent free of 15%"
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Warning",
			"statusRating": 2
		},
		"statusText": "Warning",
		"statusRating": 2
	}

In these example the **applicationInfo**, and **uptime** properties are examples that might be used in an actual implementation. Application developers are likely to choose different names and/or report different information about the server. Developers should focus on features of the server, operating system, and application that are critical to reliable operation.

# Level 2 Diagnostic: Configuration

A level two diagnostic tests whether required configuration is in place and provides configuration values as appropriate.

**Request**

    http://servicename.org/status?level=2

**Success Response**

	{
		"unit": "Student",
		"level": 2,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Ideal",
		"statusRating": 4
	}

**Failure Response**

	{
		"unit": "Student",
		"level": 2,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [],
					"statusText": "Failed",
					"statusRating": 0
				}
			],
			"statusText": "Failed",
			"statusRating": 0,
			error: "Unable to retrieve configuration from ProgMan."
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				url: "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Failed",
		"statusRating": 0
	}

In these examples the **environmentSettings** and **progmanConfigurations** properties are examples that might be used in an actual implementation. Application developers may choose to use different names and/or report different information. Care should be taken not to disclose sensitive configuration information (see Security Considerations below).   A value of "\<redacted\>" SHOULD be used to denote the information is being displayed due to security considerations.


# Level 3 Diagnostic: Read Data

A level three diagnostic tests whether the system can read from the database. If the server is directly connected to a database, it should perform a read operation on that database. If it's connected by way of an intermediary service or API, it would perform a read operation through that API or service.

**Request**

    http://servicename.org/status?level=3

**Success Response**

	{
		"unit": "Student",
		"level": 3,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 2,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Config",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 3,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Session",
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Ideal",
		"statusRating": 4
	}
	
**Failure Response**

	{
		"unit": "Student",
		"level": 3,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
							{
							"type": "READ",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						}
					],
					"schema": "Config",
					"statusText": "Failed",
					"statusRating": 0
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						}
					],
					"schema": "Session",
					"statusText": "Failed",
					"statusRating": 0
				}
			],
			"statusText": "Failed",
			"statusRating": 0
		},
		"statusText": "Failed",
		"statusRating": 0
	}

In these examples the **repostories** and **databaseOperations** properties are an example that might be used in an actual implementation. Application developers may choose to use different names and/or report different information.

# Level 4 Diagnostic: Write Data

A level four diagnostic tests whether the system can write data to the database and whether it can read the data back with fidelity. If the server is directly connected to a database, it SHOULD perform the write and read operations on that database. If it's connected by way of an intermediary service or API, it SHOULD perform a write and read through that API or service.

The write and read should be performed to an area of the database that exercises all of the communications and security provisions but does not affect the fidelity of actual data.

**Request**

    http://servicename.org/status?level=4

**Success Response**

	{
		"unit": "Student",
		"level": 4,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 2,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 5,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Config",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 3,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 7,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Session",
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Ideal",
		"statusRating": 4
	}
	
**Failure Response**

	{
		"unit": "Student",
		"level": 4,
		"dateTime": "Wed Nov 30 22:30:27 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Wed Nov 30 22:30:27 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						},
						{
							"type": "WRITE",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						}
					],
					"schema": "Config",
					"statusText": "Failed",
					"statusRating": 0
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						},
						{
							"type": "WRITE",
							"responseTime": null,
							"statusText": "Failed",
							"statusRating": 0,
							"error": "Connection timeout trying to read from the database"
						}
					],
					"schema": "Session",
					"statusText": "Failed",
					"statusRating": 0
				}
			],
			"statusText": "Failed",
			"statusRating": 0
		},
		"statusText": "Failed",
		"statusRating": 0
	}

In these examples the **repostories** and **databaseOperations** properties are an example that might be used in an actual implementation. Application developers may choose to use different names and/or report different information.

# Level 5 Diagnostic: Dependencies

A level 5 diagnostic tests all of the providers on which the service depends. In an ideal case, those providers also implement the diagnostic API in which case this diagnostic invokes the corresponding diagnostics on the provider services and embeds the results. In many cases, they use other APIs and so appropriate, benign operations should be invoked.

**Request**

    http://servicename.org/status?level=5

**Success Response**

	{
		"unit": "Student",
		"level": 5,
		"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Fri Dec 02 21:49:07 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 2,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 5,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Config",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 3,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 7,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Session",
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"providers": {
			"statuses": [
				{
					"unit": "ART",
					"level": 0,
					"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "Progman",
					"level": 0,
					"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "EquationScoring",
					"level": 0,
					"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "StudentReportProcessor",
					"level": 0,
					"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "TIS",
					"level": 0,
					"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"statusText": "Ideal",
		"statusRating": 4
	}

**Warning Response**

	{
		"unit": "Student",
		"level": 5,
		"dateTime": "Fri Dec 02 21:49:07 UTC 2016",
		"configuration": {
			"environmentSettings": [
				{
					"name": "progman.locator",
					"value": "tds,Development"
				},
				{
					"name": "progman.baseUri",
					"value": "http://progman.servicename.org/rest/"
				},
				{
					"name": "spring.profiles.active",
					"value": "mna.client.null,progman.client.impl.integration,server.singleinstance"
				}
			],
			"progmanConfigurations": [
				{
					"name": "tds",
					"envName": "Development",
					"properties": [
						{
							"name": "setting1",
							"value": "value1"
						},
						{
							"name": "secureSetting",
							"value": "<redacted>"
						}
					],
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"localSystem": {
			"javaProcessCpuLoad": 0.011350293542074364,
			"systemCpuLoad": 0.009700866648029074,
			"totalPhysicalMemory": 2097680384,
			"freePhysicalMemory": 68591616,
			"jvmTotalMemory": 384110592,
			"jvmFreeMemory": 149964664,
			"jvmMaxMemory": 528154624,
			"volumes": [
				{
					"absolutePath": "/",
					"totalSpace": 31562334208,
					"freeSpace": 19273670656,
					"percentFree": 61.06541591310686,
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"applicationInfo": {
				"url": "server1.servicename.org",
				"version": "2015.3.1.17178"
			},
			"uptime": {
				"serverTime": "Fri Dec 02 21:49:07 UTC 2016",
				"runningSince": "Wed Jan 12 07:12:38 UTC 2016",
				"runningFor": "11DT23H50M37S"
			},
			"statusText": "Ideal",
			"statusRating": 4
		},
		"database": {
			"repositories": [
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 2,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 5,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Config",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"databaseOperations": [
						{
							"type": "READ",
							"responseTime": 3,
							"statusText": "Ideal",
							"statusRating": 4
						},
						{
							"type": "WRITE",
							"responseTime": 7,
							"statusText": "Ideal",
							"statusRating": 4
						}
					],
					"schema": "Session",
					"statusText": "Ideal",
					"statusRating": 4
				}
			],
			"statusText": "Ideal",
			"statusRating": 4
		},
		"providers": {
			"statuses": [
				{
					"unit": "ART",
					"level": 0,
					"dateTime": "Fri Dec 02 21:53:23 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "Progman",
					"level": 0,
					"dateTime": "Fri Dec 02 21:53:23 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "EquationScoring",
					"level": 0,
					"dateTime": "Fri Dec 02 21:53:23 UTC 2016",
					"statusText": "Ideal",
					"statusRating": 4
				},
				{
					"unit": "StudentReportProcessor",
					"level": 0,
					"dateTime": "Fri Dec 02 21:53:23 UTC 2016",
					"statusText": "Warning",
					"statusRating": 2,
					"warning": "StudentReportProcessor service is not running.",
				},
				{
					"unit": "TIS",
					"level": 0,
					"dateTime": "Fri Dec 02 21:53:23 UTC 2016",
					"statusText": "Warning",
					"statusRating": 2,
					"warning": "Could not retrieve TIS url since StudentReportProcessor is not running.",
				}
			],
			"statusText": "Warning",
			"statusRating": 2
		},
		"statusText": "Warning",
		"statusRating": 2
	}

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
