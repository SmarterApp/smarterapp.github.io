---
title: Hash Student ID Sample
date: 2015-09-10
docurl: https://github.com/SmarterApp/HashStudentIdSample
status: Sample
---
Sample code to demonstrate how to hash a StudentID into an AlternateSSID suitable for use in de-identified student data. The code is written in C# for the Microsoft.NET platform and makes use of the associated cryptographic library. Other platforms have equivalent libraries. For example, [here is a sample provided by Amazon in Java](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AuthJavaSampleHMACSignature.html).

Vendors are not required to use this method for generating the AlternateSSID but it is a recommended practice.