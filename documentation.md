---
layout: page
title: Documentation
permalink: /docs/
---

<small class="pull-right">Revision 11 -- Updated 30 July 2015</small>

**TABLE OF CONTENTS**

<!-- TOC depthFrom:2 depthTo:6 withLinks:1 updateOnSave:0 orderedList:0 -->

- [1. Introduction](#introduction)
	- [1.1 Overview](#overview)
	- [1.2 Architectural Overview](#architectural-overview)
	- [1.3 API Versioning](#api-versioning)
		- [1.3.1 Current Version](#current-version)
		- [1.3.2 API Evolution - Non Breaking Changes](#api-evolution-non-breaking-changes)
	- [1.3.3 API Evolution - Breaking Changes](#api-evolution-breaking-changes)
- [2. Authentication](#authentication)
	- [2.1 Overview](#overview)
	- [2.2 Setup](#setup)
	- [2.3 URLs](#urls)
	- [2.4 The Security Token](#the-security-token)
	- [2.5 Creating the Security Token in .NET](#creating-the-security-token-in-net)
	- [2.6 Creating the Security Token on Other Platforms](#creating-the-security-token-on-other-platforms)
	- [2.7 Building the final URL](#building-the-final-url)
	- [2.8 Verifying a Security Token](#verifying-a-security-token)
- [3. Client Types](#client-types)
	- [3.1 Client Web Sites](#client-web-sites)
		- [3.1.1 View Listing Endpoint](#view-listing-endpoint)
	- [3.2 Portals](#portals)
- [4. API Overview](#api-overview)
	- [4.1 Overview of the Synchronisation API](#overview-of-the-synchronisation-api)
	- [4.2 Overview of the Enquiries API](#overview-of-the-enquiries-api)
	- [4.3 Conventions Used in the Web Method Specification](#conventions-used-in-the-web-method-specification)
- [5. Web Methods: Synchronisation](#web-methods-synchronisation)
	- [5.1 Request Snapshot](#request-snapshot)
		- [5.1.1 Overview](#overview)

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

| Parameter   | Description                                                                                                      |
|:------------|:-----------------------------------------------------------------------------------------------------------------|
| `ClientID`  | The ClientID supplied during the Setup procedure.                                                                |
| `TimeStamp` | The current time as reported by the client’s computer as a UTC (GMT+0) date and time down to the current minute. |
| `Salt`      | a 64-bit number randomly generated on the client’s PC.                                                           |
| `Digest`    | a Base64 conversion of an SHA1 computed hash of the password and the above data.                                 |

### 2.5 Creating the Security Token in .NET

Fusion has .NET C# source code that will generate a SecurityToken and export it in a format ready to be appended to an URL.  Contact Fusion should you wish to make use of this source code.

### 2.6 Creating the Security Token on Other Platforms

The following rules describe how to generate a Security Token from first principles:

- `ClientID` – this number is supplied by Fusion.
- `TimeStamp` – generate a text string in the following format (MM, DD, HH and MM are 0-padded) and HH is in 24-hour format
  - YYYY-MM-DD-HH-MM
- `Salt` – generate a 64-bit number as a base-10 ASCII string.  We suggest using the computers TickCount as an easy random number
- `Digest` – see below

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

    &clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793

Appended to the base URL

    http://staging-feedstore.fusionagency.net/v1/sync/RequestSnapshot/?clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793

_Please be aware that a GET request to a method that expects a POST will return 404 even though the endpoint exists._

### 2.8 Verifying a Security Token

The NotifyChangesAvailable web method call (see API section later) supplies a Security Token on the URL.  The client needs to verify that the security token is valid, otherwise fake sites could spoof calls into this endpoint.

Verifying a token is a simple process:

- Generate a digest using the client’s ClientID and Password together with the TimeStamp and Salt supplied in the URL call (as described in the section above on Creating a Security Token).
- Compare the generated digest with the digest supplied in the URL call.
- If the digests match, the token is valid.

## 3. Client Types

### 3.1 Client Web Sites

Each Fusion client has the option of using a Fusion Front-end web-site based on a standard Fusion template, or building a custom web site.

A custom web site may be linked to a single Fusion Office, or it may aggregate a number of Offices together.

#### 3.1.1 View Listing Endpoint

Fusion exposes a link inside its user interface to navigate directly to a listing’s details page on the client’s web site.  Each custom web site will need to supply Fusion with a fixed URL that will redirect to the listings page.  The URL should be:

    http://some.web.site/some/path?clientId=12&listingId=456

The custom web site developers must implement this path, and supply the URL part:


The custom web site developers must implement this path, and supply the URL part:

    http://some.web.site/some/path

Fusion will append the parameter part:

    ?clientId=12&listingId=456

The parameters are defined as:

- The `clientId` will be the same clientId given to the custom web site and used as a handle for accessing the Sync Store.
- The `listingId` is the Fusion listingId as returned in the `<Listing>` element.

### 3.2 Portals

A portal client aggregates listings from a number of different Agencies and Offices.

## 4. API Overview

### 4.1 Overview of the Synchronisation API

The Fusion Sync Store exposes an interface that allows a client to maintain a copy of the Fusion database, getting updates in an asynchronous, event-driven, always-up-to-date manner that makes optimal use of bandwidth.

The system is built around a stream of Create, Update and Delete change events for which the client can poll.

The Sync Store also supports a notification system.  The Sync Store will call the client when new changes are available.  The client can then poll the Sync Store to retrieve these changes.  In this way, the simple architecture of a polled system is converted into an efficient asynchronous eventing system.

Since the Sync Store works on a stream of events, it makes the assumption that its database is consistent with the client’s database **before any given event is processed**.  In an ideal world, this will always be the case.  Unfortunately, we do not live in an ideal world, and various issues can cause the two databases to get out of sync: network failures, database corruptions, programming bugs, etc.

The Sync Store implements a snapshot mechanism which allows it to resend the state of its various objects to the client to ensure a common baseline state is achieved.  This snapshot system is also used to populate an empty database.  The client may request a snapshot, and the Sync Store itself may issue a snapshot.  For this latter reason, all clients MUST implement their side of the snapshot protocol.

The snapshot send is a heavy-weight process that resends all objects in the system.  This could be a large dataset for clients with access to a vast number of listings.  In some cases, the client may have a recent “last known good state” of its database (perhaps from a backup, or some snapshot state).  In cases like these, the client can restore its database state to its last known good state and then request the Sync Store to “resend” all change events from the restored date-time to the present time.  For large databases, this will be a much faster synchronisation mechanism than a full database restore via a snapshot.

The Sync Store implements a method that provides direct access to a single listing.  The client can make a request to receive the current state of a listing (or a deleted state if the listing does not exist).  This can be used as a super lightweight means of synchronising a few listings that are known to have become corrupted.

The Sync Store runs a housekeeping background process at a pre-determined time in the early morning.  While the housekeeping process is running, a number of the web method calls will fail with a Housekeeping In Progress exception.  If that happens, the client should back off and try again later.

### 4.2 Overview of the Enquiries API

The Fusion Sync Store exposes an interface that allows any client to send an “enquiry” generated by a third party user of the client’s system, into the Fusion system.  When received, the Fusion system will route the enquiry to the appropriate agency office for processing.

Each enquiry must include the user’s name, email address and message body text.  An enquiry may also optionally include the user’s mobile telephone number and a summary subject line.

If the enquiry relates to a specific listing, either the Listing ID or the Fusion Reference should be included in the enquiry.

If the enquiry is of a general nature (i.e. the enquiry is not related to a specific listing), the target Office ID should be included in the enquiry.

### 4.3 Conventions Used in the Web Method Specification

- The `|` symbol defines the options that the relevant attribute can take on.  Only one of the options is allowed per attribute.
- In the API Reference section, when an attribute is stated to be _**optional**_ it means that either the attribute will be missing, _**or**_ the attribute value will be an empty string.

## 5. Web Methods: Synchronisation

### 5.1 Request Snapshot

#### 5.1.1 Overview

- A client calls this to initiate a Snapshot resend of **all object types.**
- The next call to the `GetChanges` method will result in the current change queue being paused, while a full Snapshot stream of all object types is sent.  After the snapshot has been sent, any queued changes will be sent.
- If this is called while another Snapshot or Rollback is in progress, that Snapshot/Rollback will be aborted and this new Snapshot will start immediately.
- Calling this method will raise a `NotifyChangesAvailable` event.  This allows the client code to be written asynchronously and call `GetChanges` only on receipt of the Notify event.
- For further details on Snapshot, see the `GetChanges` method.
