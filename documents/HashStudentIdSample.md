---
title: Student ID Cryptographic Hash Algorithm
dateCreated: 2015-10-02
date: 2015-10-02
version: 1.0
layout: document
author: Brandt Redd
---
Each year members submit to Smarter Balanced their student test results and student responses to items. The minimum requirement is for de-identified data; members may elect to send identified data. Smarter Balanced uses these data to maintain and improve the assessments including calibrating new items and producing technical reports.

When submitting de-identified data, members should include in the data an Alternate State Student ID (AlternateSSID) that uniquely identifies each student and is consistent from year to year. This alternate ID will be used in future years to compare student performance with previous years’ data and to measure growth.

To preserve student privacy, it should not be possible for anyone to derive the State Student ID (SSID) from the AlternateSSID. Smarter Balanced has recommended the use of the HMAC-SHA1 cryptographic hashing algorithm for this purpose. The algorithm can be applied in various ways that are all equally effective in terms of generating consistent AlternateSSIDs and preserving student privacy. A small variation in implementation, however, will result in different AlternateSSIDs. This document and the [accompanying source code sample](https://github.com/SmarterApp/HashStudentIdSample) describe one effective application of the algorithm. It is not a requirement but following this recommendation will result in consistent IDs year over year.

## Keyed Cryptographic Hash

A keyed cryptographic hash function accepts a string of data (the State Student ID) and a secret key. From these inputs it produces a hash code. This can be used for the AlternateSSID in de-identified data sets.

![Keyed Cryptographic Hash](keyedcryptographichash.png)

The same SSID and secret key will always generate the same AlternateSSID. However, it is computationally impractical to reverse the hash and gain the SSID back, even if the secret key is known. Thus, an entity would have to have both the secret key and the entire roster of all unencrypted student IDs to be able to match a de-identified record to a student ID.

## Implementation

Most contemporary programming environments have a cryptographic library with an implementation of the HMAC-SHA1 algorithm. The [HMAC algorithm is described in RFC 2104](https://tools.ietf.org/html/rfc2104) and the [SHA-1 algorithm is described in RFC 3174](https://tools.ietf.org/html/rfc3174).

For this application both the secret key and the SSID are assumed to be [Unicode](http://unicode.org/) strings of arbitrary length. Here are the steps to the recommended application:

 1. Remove leading and trailing [whitespace](https://en.wikipedia.org/wiki/Whitespace_character) characters from the secret key.
 2. Encode the secret key as a series of bytes using [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding. A [byte-order mark](https://en.wikipedia.org/wiki/Byte_order_mark) *must not* be present. (Note that certain libraries such as the [Microsoft .NET Framework](https://en.wikipedia.org/wiki/.NET_Framework) include the byte-order mark by default and this must be suppressed.) 
 3. Hash the secret key using the SHA-1 algorithm. Any [terminating null](https://en.wikipedia.org/wiki/Null_character) character must not be included in the data being hashed. This results in a 160-bit byte value in the form of a 20-byte array. This will be used as the binary key.
 4. Remove leading and trailing whitespace characters from the State Student ID (SSID).
 5. Encode the SSID as a series of bytes using UTF-8 encoding. A byte-order mark *must not* be present and a terminating null should not be included.
 6. Apply the HMAC-SHA1 keyed-hash algorithm to the encoded SSID using the binary key from step 3. This results in a 160-bit binary value in the form of a 20-byte array.
 7. Convert the hash into a 40-character upper-case hexadecimal string.

## Sample Code

Smarter Balanced has built an open source sample implemented in Microsoft C#. [It is available here](https://github.com/SmarterApp/HashStudentIdSample). It makes use of the Microsoft cryptographic library. Other platforms have equivalent libraries. For example, [here is a similar algorithm provided by Amazon written in Java](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AuthJavaSampleHMACSignature.html).

## Validation Data

The following data may be used to determine that an implementation is consistent with this description and the sample code. Other approaches may be equally secure but will not match these results.

SSID|Secret Key|AlternateSSID
----|----------|-------------
39IJH43982|OurStudentsSucceed|56F8F15D4B19A1DB3A884745103A9A92A845E225
BB-8|The Force Awakens|9F5685FB73F7315EA0707202F1B54FAC973875B3
42|Slartibartfast|87BD175DFC231FE7E2D2030C8A6D0520AC629083
7401203|Maher-shalal-hash-baz|77D015E4EA3CC9DB4EBAE093954CBC805D55013C
