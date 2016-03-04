---
layout: page
title: Documentation
permalink: /docs/
---

<small class="pull-right">Revision 11 -- Updated 30 July 2015</small>

**TABLE OF CONTENTS**

<!-- TOC depthFrom:2 depthTo:4 withLinks:1 updateOnSave:1 orderedList:0 -->

- [1. Introduction](#1-introduction)
	- [1.1 Overview](#11-overview)
	- [1.2 Architectural Overview](#12-architectural-overview)
	- [1.3 API Versioning](#13-api-versioning)
		- [1.3.1 Current Version](#131-current-version)
		- [1.3.2 API Evolution - Non Breaking Changes](#132-api-evolution-non-breaking-changes)
	- [1.3.3 API Evolution - Breaking Changes](#133-api-evolution-breaking-changes)
- [2. Authentication](#2-authentication)
	- [2.1 Overview](#21-overview)
	- [2.2 Setup](#22-setup)
	- [2.3 URLs](#23-urls)
	- [2.4 The Security Token](#24-the-security-token)
	- [2.5 Creating the Security Token in .NET](#25-creating-the-security-token-in-net)
	- [2.6 Creating the Security Token on Other Platforms](#26-creating-the-security-token-on-other-platforms)
	- [2.7 Building the final URL](#27-building-the-final-url)
	- [2.8 Verifying a Security Token](#28-verifying-a-security-token)

<!-- /TOC -->

## 1. Introduction

### 1.1 Overview

This document describes the API for accessing the Fusion Sync Store.

### 1.2 Architectural Overview

The Fusion Sync Store exposes an HTTP endpoint.

Clients request information via that endpoint using a standard request-response web method pattern, as detailed within this document.

Every client is linked to a collection of Fusion Offices.

The client calls the Sync Store API to incrementally retrieve create, update and delete change events for all offices, agents, developments, listings and suburbs.

### 1.3 API Versioning

#### 1.3.1 Current Version

* The API is currently on a **major version** of **v1**.

#### 1.3.2 API Evolution - Non Breaking Changes

* Additional XML attributes and elements may be _**added**_ to the existing data structures in the future _**without**_ changing the protocol version number.  i.e. your client code must be prepared to deal with new attributes and Elements outside those listed in this document.  However, _**existing**_ attributes and elements will _**not**_ change.

* Additional web methods may be added over time to the same protocol version number.

* Additional _**optional**_ parameters may be added to existing web methods over time to the same protocol version number.

* Additional exception types may be added over time to the same protocol version number.

### 1.3.3 API Evolution - Breaking Changes

* If there is requirement to **change** any existing url parameter or **change** any attribute or element in a web method call or XML return structure, a new Protocol Version number will be created, and the previous version of the protocol will be maintained for backwards compatibility purposes for a period of time.

* Should an old protocol be deprecated, all clients of that version will be notified ahead of time and an alternative protocol provided.  The client will be allocated a period of time to transition to the new protocol, before the old version is discontinued.

## 2. Authentication

### 2.1 Overview

Every web method takes a mandatory SecurityToken as a parameter.  Each time a client calls a method, it needs to re-create this security token.

### 2.2 Setup

Every client must request a ClientID and Password from our support team.
The Password must be kept secure at all times.

### 2.3 URLs

- Testing: http://staging-feedstore.fusionagency.net/v1/sync/
- Production: http://za-feedstore.fusionagency.net/v1/sync/

All development and testing is to take place against the staging URL.

### 2.4 The Security Token

The SecurityToken consists of these 4 url parameters:
- ClientID - The ClientID supplied during the Setup procedure.
- TimeStamp – The current time as reported by the client’s computer as a UTC (GMT+0) date and time down to the current minute.
- Salt – a 64-bit number randomly generated on the client’s PC.
- Digest – a Base64 conversion of an SHA1 computed hash of the password and the above data.

### 2.5 Creating the Security Token in .NET

Fusion has .NET C# source code that will generate a SecurityToken and export it in a format ready to be appended to an URL.  Contact Fusion should you wish to make use of this source code.

### 2.6 Creating the Security Token on Other Platforms

The following rules describe how to generate a Security Token from first principles:

- **ClientID** – this number is supplied by Fusion.
- **TimeStamp** – generate a text string in the following format (MM, DD, HH and MM are 0-padded) and HH is in 24-hour format
  - YYYY-MM-DD-HH-MM
- **Salt** – generate a 64-bit number as a base-10 ASCII string.  We suggest using the computers TickCount as an easy random number
- **Digest** – see below

The Digest is calculated using the following **pseudo-code** algorithm:

```
KeyString = TimeStamp + “*” + Password + “*” + Salt;
Utf8String = ConvertToUtf8( KeyString )
ByteArray = ConvertToByteArray( Utf8String )
Hash = SHA1Hash( ByteArray )
Digest = Base64Encoder( Hash )
UrlFriendlyDigest = UrlEncode( Digest )
```

### 2.7 Building the final URL

Every Web Method call should have the security token parameters appended to it.

The following gives an example of a security token encoded in URL format:

```
&clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793
```

Appended to the base URL

```
http://staging-feedstore.fusionagency.net/v1/sync/RequestSnapshot/?clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793
```

_Please be aware that a GET request to a method that expects a POST will return 404 even though the endpoint exists._

### 2.8 Verifying a Security Token

The NotifyChangesAvailable web method call (see API section later) supplies a Security Token on the URL.  The client needs to verify that the security token is valid, otherwise fake sites could spoof calls into this endpoint.

Verifying a token is a simple process:

- Generate a digest using the client’s ClientID and Password together with the TimeStamp and Salt supplied in the URL call (as described in the section above on Creating a Security Token).
- Compare the generated digest with the digest supplied in the URL call.
- If the digests match, the token is valid.
