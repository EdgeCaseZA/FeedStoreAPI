---
layout: page
title: Documentation
permalink: /docs/
---

<small class="pull-right">Revision 11 -- Updated 30 July 2015</small>

**TABLE OF CONTENTS**

<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:0 orderedList:0 -->

- [1. Introduction](#1-introduction)
	- [1.1 Overview](#11-overview)
	- [1.2 Architectural Overview](#12-architectural-overview)
	- [1.3 API Versioning](#13-api-versioning)
- [2. Authentication](#2-authentication)
	- [2.1 Overview](#21-overview)
	- [2.2 Setup](#22-setup)
	- [2.3 URLs](#23-urls)
	- [2.4 The Security Token](#24-the-security-token)
	- [2.5 Creating the Security Token in .NET](#25-creating-the-security-token-in-net)
	- [2.6 Creating the Security Token on Other Platforms](#26-creating-the-security-token-on-other-platforms)
	- [2.7 Building the final URL](#27-building-the-final-url)
	- [2.8 Verifying a Security Token](#28-verifying-a-security-token)
- [3. Client Types](#3-client-types)
	- [3.1 Client Web Sites](#31-client-web-sites)
	- [3.2 Portals](#32-portals)
- [4. API Overview](#4-api-overview)
	- [4.1 Overview of the Synchronisation API](#41-overview-of-the-synchronisation-api)
	- [4.2 Overview of the Enquiries API](#42-overview-of-the-enquiries-api)
	- [4.3 Conventions Used in the Web Method Specification](#43-conventions-used-in-the-web-method-specification)
- [5. Web Methods: Synchronisation](#5-web-methods-synchronisation)
	- [5.1 Request Snapshot](#51-request-snapshot)
	- [5.2 Request Rollback](#52-request-rollback)
	- [5.3 Request Listing](#53-request-listing)
	- [5.4 Get Changes](#54-get-changes)
	- [5.5 Notify Changes Available](#55-notify-changes-available)
- [6 Web Methods: Enquiries](#6-web-methods-enquiries)
	- [6.1 Send Enquiry](#61-send-enquiry)
- [7. APPENDIX A: SYNC OBJECT TYPES](#7-appendix-a-sync-object-types)
	- [7.1 Agent](#71-agent)
	- [7.2 Office](72-office)

<!-- /TOC -->

## 1. Introduction <a id="1-introduction"></a>

### 1.1 Overview <a id="11-overview"></a>

This document describes the API for accessing the Fusion Sync Store.

### 1.2 Architectural Overview <a id="12-architectural-overview"></a>

The Fusion Sync Store exposes an HTTP endpoint.

Clients request information via that endpoint using a standard request-response
web method pattern, as detailed within this document.

Every client is linked to a collection of Fusion Offices.

The client calls the Sync Store API to incrementally retrieve create, update and
delete change events for all offices, agents, developments, listings and suburbs.

### 1.3 API Versioning <a id="13-api-versioning"></a>

#### 1.3.1 Current Version

- The API is currently on a **major version** of **v1**.

#### 1.3.2 API Evolution - Non Breaking Changes

- Additional XML attributes and elements may be _**added**_ to the existing data
  structures in the future _**without**_ changing the protocol version number.  
	i.e. your client code must be prepared to deal with new attributes and
	Elements outside those listed in this document.  However, _**existing**_
	attributes and elements will _**not**_ change.

- Additional web methods may be added over time to the same protocol version
  number.

- Additional _**optional**_ parameters may be added to existing web methods over
  time to the same protocol version number.

- Additional exception types may be added over time to the same protocol version
  number.

#### 1.3.3 API Evolution - Breaking Changes

- If there is requirement to **change** any existing url parameter or **change**
  any attribute or element in a web method call or XML return structure, a new
	Protocol Version number will be created, and the previous version of the
	protocol will be maintained for backwards compatibility purposes for a period
	of time.

- Should an old protocol be deprecated, all clients of that version will be
  notified ahead of time and an alternative protocol provided.  The client will
	be allocated a period of time to transition to the new protocol, before the
	old version is discontinued.

## 2. Authentication <a id="2-authentication"></a>

### 2.1 Overview <a id="21-overview"></a>

Every web method takes a mandatory SecurityToken as a parameter.  Each time a
client calls a method, it needs to re-create this security token.

### 2.2 Setup <a id="22-setup"></a>

Every client must request a ClientID and Password from our support team.
The Password must be kept secure at all times.

### 2.3 URLs <a id="23-urls"></a>

- Testing: [http://staging-feedstore.fusionagency.net/v1/sync/](http://staging-feedstore.fusionagency.net/v1/sync/)
- Production: [http://za-feedstore.fusionagency.net/v1/sync/](http://za-feedstore.fusionagency.net/v1/sync/)

All development and testing is to take place against the staging URL.

### 2.4 The Security Token <a id="24-the-security-token"></a>

The SecurityToken consists of these 4 url parameters:

| Parameter   | Description                                                                                                      |
|:------------|:-----------------------------------------------------------------------------------------------------------------|
| `ClientID`  | The ClientID supplied during the Setup procedure.                                                                |
| `TimeStamp` | The current time as reported by the client’s computer as a UTC (GMT+0) date and time down to the current minute. |
| `Salt`      | a 64-bit number randomly generated on the client’s PC.                                                           |
| `Digest`    | a Base64 conversion of an SHA1 computed hash of the password and the above data.                                 |

### 2.5 Creating the Security Token in .NET <a id="25-creating-the-security-token-in-net"></a>

Fusion has .NET C# source code that will generate a SecurityToken and export it
in a format ready to be appended to an URL.  Contact Fusion should you wish to
make use of this source code.

### 2.6 Creating the Security Token on Other Platforms <a id="26-creating-the-security-token-on-other-platforms"></a>

The following rules describe how to generate a Security Token from first
principles:

- `ClientID` – this number is supplied by Fusion.
- `TimeStamp` – generate a text string in the following format (MM, DD, HH and
	MM are 0-padded) and HH is in 24-hour format
  - YYYY-MM-DD-HH-MM
- `Salt` – generate a 64-bit number as a base-10 ASCII string.  We suggest using
  the computers TickCount as an easy random number
- `Digest` – see below

The Digest is calculated using the following **pseudo-code** algorithm:

	KeyString = TimeStamp + “*" + Password + “*" + Salt;
	Utf8String = ConvertToUtf8( KeyString )
	ByteArray = ConvertToByteArray( Utf8String )
	Hash = SHA1Hash( ByteArray )
	Digest = Base64Encoder( Hash )
	UrlFriendlyDigest = UrlEncode( Digest )

### 2.7 Building the final URL <a id="27-building-the-final-url"></a>

Every Web Method call should have the security token parameters appended to it.

The following gives an example of a security token encoded in URL format:

    &clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793

Appended to the base URL

    http://staging-feedstore.fusionagency.net/v1/sync/RequestSnapshot/?clientId=5&timeStamp=2011-12-03-22-05&salt=23872387232&digest=623fa5af23793

_Please be aware that a GET request to a method that expects a POST will return
404 even though the endpoint exists._

### 2.8 Verifying a Security Token <a id="28-verifying-a-security-token"></a>

The NotifyChangesAvailable web method call (see API section later) supplies a
Security Token on the URL.  The client needs to verify that the security token
is valid, otherwise fake sites could spoof calls into this endpoint.

Verifying a token is a simple process:

- Generate a digest using the client’s ClientID and Password together with the
  TimeStamp and Salt supplied in the URL call (as described in the section above
	on Creating a Security Token).
- Compare the generated digest with the digest supplied in the URL call.
- If the digests match, the token is valid.

## 3. Client Types <a id="3-client-types"></a>

### 3.1 Client Web Sites <a id="31-client-web-sites"></a>

Each Fusion client has the option of using a Fusion Front-end web-site based on
a standard Fusion template, or building a custom web site.

A custom web site may be linked to a single Fusion Office, or it may aggregate
a number of Offices together.

#### 3.1.1 View Listing Endpoint

Fusion exposes a link inside its user interface to navigate directly to a
listing’s details page on the client’s web site.  Each custom web site will need
to supply Fusion with a fixed URL that will redirect to the listings page.
The URL should be:

    http://some.web.site/some/path?clientId=12&listingId=456

The custom web site developers must implement this path, and supply the URL part:


The custom web site developers must implement this path, and supply the URL part:

    http://some.web.site/some/path

Fusion will append the parameter part:

    ?clientId=12&listingId=456

The parameters are defined as:

- The `clientId` will be the same clientId given to the custom web site and used
as a handle for accessing the Sync Store.
- The `listingId` is the Fusion listingId as returned in the `<Listing>` element.

### 3.2 Portals <a id="32-portals"></a>

A portal client aggregates listings from a number of different Agencies and Offices.

## 4. API Overview <a id="4-api-overview"></a>

### 4.1 Overview of the Synchronisation API <a id="41-overview-of-the-synchronisation-api"></a>

The Fusion Sync Store exposes an interface that allows a client to maintain a
copy of the Fusion database, getting updates in an asynchronous, event-driven,
always-up-to-date manner that makes optimal use of bandwidth.

The system is built around a stream of Create, Update and Delete change events
for which the client can poll.

The Sync Store also supports a notification system.  The Sync Store will call
the client when new changes are available.  The client can then poll the
Sync Store to retrieve these changes.  In this way, the simple architecture of a
polled system is converted into an efficient asynchronous eventing system.

Since the Sync Store works on a stream of events, it makes the assumption that
its database is consistent with the client’s database **before any given event
is processed**.  In an ideal world, this will always be the case.  Unfortunately,
we do not live in an ideal world, and various issues can cause the two databases
to get out of sync: network failures, database corruptions, programming bugs, etc.

The Sync Store implements a snapshot mechanism which allows it to resend the
state of its various objects to the client to ensure a common baseline state is
achieved.  This snapshot system is also used to populate an empty database.  The
client may request a snapshot, and the Sync Store itself may issue a snapshot.  
For this latter reason, all clients MUST implement their side of the snapshot
protocol.

The snapshot send is a heavy-weight process that resends all objects in the
system.  This could be a large dataset for clients with access to a vast number
of listings.  In some cases, the client may have a recent “last known good state"
of its database (perhaps from a backup, or some snapshot state).  In cases like
these, the client can restore its database state to its last known good state and
then request the Sync Store to “resend" all change events from the restored
date-time to the present time.  For large databases, this will be a much faster
synchronisation mechanism than a full database restore via a snapshot.

The Sync Store implements a method that provides direct access to a single
listing.  The client can make a request to receive the current state of a
listing (or a deleted state if the listing does not exist).  This can be used
as a super lightweight means of synchronising a few listings that are known to
have become corrupted.

The Sync Store runs a housekeeping background process at a pre-determined time
in the early morning.  While the housekeeping process is running, a number of
the web method calls will fail with a Housekeeping In Progress exception.  If
that happens, the client should back off and try again later.

### 4.2 Overview of the Enquiries API <a id="42-overview-of-the-enquiries-api"></a>

The Fusion Sync Store exposes an interface that allows any client to send an
“enquiry" generated by a third party user of the client’s system, into the
Fusion system.  When received, the Fusion system will route the enquiry to the
appropriate agency office for processing.

Each enquiry must include the user’s name, email address and message body text.
An enquiry may also optionally include the user’s mobile telephone number and a
summary subject line.

If the enquiry relates to a specific listing, either the Listing ID or the
Fusion Reference should be included in the enquiry.

If the enquiry is of a general nature (i.e. the enquiry is not related to a
specific listing), the target Office ID should be included in the enquiry.

### 4.3 Conventions Used in the Web Method Specification <a id="43-conventions-used-in-the-web-method-specification"></a>

- The `|` symbol defines the options that the relevant attribute can take on.  
  Only one of the options is allowed per attribute.
- In the API Reference section, when an attribute is stated to be _**optional**_
  it means that either the attribute will be missing, _**or**_ the attribute value
  will be an empty string.

## 5. Web Methods: Synchronisation <a id="5-web-methods-synchronisation"></a>

### 5.1 Request Snapshot <a id="51-request-snapshot"></a>

#### 5.1.1 Overview

- A client calls this to initiate a Snapshot resend of **all object types.**
- The next call to the `GetChanges` method will result in the current change
  queue being paused, while a full Snapshot stream of all object types is sent.
	After the snapshot has been sent, any queued changes will be sent.
- If this is called while another Snapshot or Rollback is in progress, that
  Snapshot/Rollback will be aborted and this new Snapshot will start immediately.
- Calling this method will raise a `NotifyChangesAvailable` event.  This allows
  the client code to be written asynchronously and call `GetChanges` only on
	receipt of the Notify event.
- For further details on Snapshot, see the `GetChanges` method.

#### 5.1.2 Web Method Specification

> Name      
> : `RequestSnapshot`
>
> HTTP Verb
> : `POST`
>
> | Parameters              | Type            |           |
> |:------------------------|:----------------|:----------|
> | (`securityTokenParams`) | (SecurityToken) | Mandatory |
> | `clientId`              | Int             | Mandatory |
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://website/v1/Sync/RequestSnapshot`

#### 5.1.3 Parameters

##### 5.1.3.1. clientId

- This parameter is mandatory.
- It must match a Client ID as setup in the Fusion system.

#### 5.1.4 Returns on Success

##### 5.1.4.1. An Example

    <RequestCompleted warning="ExistingRollbackAborted|ExistingSnapshotAborted" />

##### 5.1.4.2. Root Node

- If the snapshot request was successful, a `RequestCompleted` element will be
  returned.
- An optional `warning` attribute may be present:
  - `ExistingRollbackAborted` – A Rollback was in progress.  The Rollback was
	  aborted, and the new Snapshot was started.
  - `ExistingSnapshotAborted` – A Snapshot was in progress.  The Snapshot was
	  aborted, and the new Snapshot was started.

#### 5.1.5 Exceptions on Failure

The following common exceptions may occur (see Appendix B for details):

- Invalid Client ID
- Housekeeping In Progress
- Invalid Security Token
- Security Token Expired
- Internal Error
- Service Offline

### 5.2 Request Rollback <a id="52-request-rollback"></a>

#### 5.2.1 Overview

- A client calls this to initiate an Event Rollback.
- A Rollback is used to resend the full GetChanges event history from a recent
  time in the past to the current time.  It is used as a lightweight form of
	snapshot resend, and can be used to rebuild the client’s remote store from a
	last-known good state.
- Change Events are kept for a maximum of 7 days in the past, so any attempt to
  request a Rollback from earlier than that, or later than the present time will
	result in an Invalid Start Time exception.
- There are certain scenarios when the Sync Store may purge the Changes history.  
  In those rare cases, a No History Available exception will be returned.  If
	the client receives this exception, it will have to fall back to requesting a
	full Snapshot.
- Events for all object types (see Appendix A) will be sent during the Rollback.
- If this is called while another Snapshot or Rollback is in progress, that
  Snapshot/Rollback will be aborted and this new Rollback will start immediately.
- Calling this method will raise a NotifyChangesAvailable event.  This allows
  the client code to be written asynchronously and call GetChanges only on
	receipt of the Notify event.
- For further details on Rollbacks, see the GetChanges method.

#### 5.2.2 Web Method Specification

> Name
> : `RequestRollback`
>
> HTTP Verb
> : `POST`
>
> | Parameters              | Type            |           |
> |:------------------------|:----------------|:----------|
> | (`securityTokenParams`) | (SecurityToken) | Mandatory |
> | `clientId`              | Int             | Mandatory |
> | `startTime`             | DateTime        | Mandatory |
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://website/v1/Sync/RequestRollback`

#### 5.2.3 Parameters

##### 5.2.3.1 clientId

- This parameter is mandatory.
- It must match a Client ID as setup in the Fusion system.

##### 5.2.3.2 startTime

- This parameter is mandatory.
- The `startTime` attribute defines the starting date-time from which events
  should be sent.
- The `startTime` must be no earlier than 7 days in the past, and no later than
  the current time.
- The `startTime` must be represented in GMT+0 (**NOT** local time!).
- The `startTime` is encoded in the following format:
  - YYYY-MM-DD-HH-MM-SS

- Notes:
	- The time component is represented to a resolution of a second.
	- Each component of the startTime is a number
	- Leading zeros are used if the component value is less than 10
	- The hour (HH) component is in 24-hour format
	- A hyphen is used as a separator between all components

- An example of a startTime is:  **2012-08-12-18-09-10**
	- This represents: 12 Aug 2012, 9 minutes and 10 seconds past 6pm

#### 5.2.4 Returns on Success

##### 5.2.4.1 An Example

    <RequestCompleted warning="ExistingRollbackAborted|ExistingSnapshotAborted" />

##### 5.2.4.2 Root Node

- If the Rollback request was successful, a `RequestCompleted` element will be
  returned.
- An optional `warning` attribute may be present:
	- `ExistingRollbackAborted` – A Rollback was in progress.  The Rollback was
	  aborted, and the new Rollback was started.
	- `ExistingSnapshotAborted` – A Snapshot was in progress.  The Snapshot was
	  aborted, and the new Rollback was started.

#### 5.2.5 Exceptions on Failure

- The following method-specific exceptions may occur:
	- Invalid Start Time
	- Invalid Parameter (startTime)
	- No History Available

- The following common exceptions may occur (see [_Appendix B_](#appendix-b) for
  details):
	- Invalid Client ID
	- Housekeeping In Progress
	- Invalid Security Token
	- Security Token Expired
	- Internal Error
	- Service Offline

### 5.3 Request Listing <a id="53-request-listing"></a>

#### 5.3.1 Overview

- A client calls this method to trigger an event that returns details about a
  single listing.
- A `CreateOrUpdate` event will be APPENDED to the change queue if the listing
  exists (see GetChanges method for more details).
- A `Delete` event will be APPENDED to the change queue if the listing does NOT
  exist (see GetChanges method for more details).
- A client may use this method in an emergency to generate information about a
  specific listing in the event of a minor client database corruption.  For
  larger synchronisation problems, a Rollback or Snapshot is recommended.
- If a Snapshot or Rollback is in progress when this method is called, this
  method will still queue the listing event and return successfully.  Once the
	Snapshot or Rollback has completed, this listing event will be returned.

#### 5.3.2 Web Method Specification

> Name
> : `RequestListing`
>
> HTTP Verb
> : `POST`
>
> | Parameters              | Type            |           |
> |:------------------------|:----------------|:----------|
> | (`securityTokenParams`) | (SecurityToken) | Mandatory |
> | `clientId`              | Int             | Mandatory |
> | `listingId`             | Int             | Mandatory |
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://website/v1/Sync/RequestListing`


#### 5.3.3 Parameters

##### 5.3.3.1 clientId

- This parameter is mandatory.
- It must match a Client ID as setup in the Fusion system.

##### 5.3.3.2 listingId

- This parameter is mandatory.
- It must match a ListingId in the Fusion system, and one that the client has
  permission to access.

#### 5.3.4 Returns on Success

##### 5.3.4.1 An Example

    <RequestCompleted />

##### 5.3.4.2 Root Node

- If the request was successful, a `RequestCompleted` element will be returned.

#### 5.3.5 Exceptions on Failure

The following common exceptions may occur (see Appendix B for details):

- Invalid Parameter (listingId)
- Invalid Client ID
- Housekeeping In Progress
- Invalid Security Token
- Security Token Expired
- Internal Error
- Service Offline

### 5.4 Get Changes <a id="54-get-changes"></a>

#### 5.4.1 Overview

- This method is the primary means by which the client receives events to update
  its local database.
- The Sync Store will call the client’s Notification end-point should data appear
  in the queue.  On receipt of that call, the client should initiate the receiving
  of data from the queue by calling this method.
- The client may also choose to poll this method periodically should a
  ChangeNotificationEvent be missed by the client for some client-side reason.
- The XML return result from a single `GetChanges` call will not exceed 10MB in size.
- Since there may be many changes present in the queue, this call may only
  return a subset of the events waiting in the queue. While events are being
	returned, the client should continue calling this method to check if any more
	events are present.
- This call is designed to be invoked from a single thread of execution.  A
  client should never invoke this method from more than one thread at a time.
- This method can return data to satisfy one of these three use-cases:
	- Standard Change Events – these are sent in the normal course of operation as
	  the system data objects change over time.
	- Snapshot Events – these are sent if a snapshot was requested by the client,
	  or the Sync Store decided a snapshot needed to be sent.
	- Rollback Events – these are sent if a rollback was requested by the client.

#### 5.4.2 Web Method Specification

> Name
> : `GetChanges`
>
> HTTP Verb
> : `POST`
>
> | Parameters            | Type            |           |
> |:----------------------|:----------------|:----------|
> | (`securityTokenParams`) | (SecurityToken) | Mandatory |
> | `clientId`              | Int             | Mandatory |
> | `commitToken`           | String          | Mandatory |
>
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://website/v1/Sync/GetChanges`

#### 5.4.3 Parameters

##### 5.4.3.1 clientId

- This parameter is optional.
- If `commitToken` is supplied, it’s value should be a copy of the `commitToken`
  that was returned in the previous call to `GetChanges`.  Passing this token
	back to the Sync Store serves to inform the Sync Store that the previous set
	of changes sent to the client have been successfully applied to its local
	database.  The 	Sync Store will then return the next set of changes in the
	queue.
- If the token is not present, the Sync Store will resend the last set of
  changes.  If this is the first time the client has called `GetChanges`, this
	call will return the initial set of changes in the queue.

#### 5.4.4 Returns on Success

##### 5.4.4.1 Standard Change Events Example

	<Changes clientId="12" commitToken="7dsg367d3n89y3">

	   <CreateOrUpdate>
	      <Office ... />
	   </CreateOrUpdate>

	   <CreateOrUpdate>
	      <Agent ... />
	   </CreateOrUpdate>

	   <Delete>
	      <OfficeRef id="528" />
	   </Delete>

	   <Delete>
	      <ListingRef id="955" />
	   </Delete>

	</Changes>

##### 5.4.4.2 Snapshot Example

	<Changes clientId="12" commitToken="7dsg367d3n89y3">

	   <BeginSnapshot types="Offices,Agents,Developments,Listings,AreaTree" />
	   <Snapshot> ... </Snapshot>
	   <Snapshot> ... </Snapshot>
	   <Snapshot> ... </Snapshot>
	   <EndSnapshot />

	</Changes>

##### 5.4.4.3 Rollback Example

	<Changes clientId="12" commitToken="7dsg367d3n89y3">

	   <Rollback to="2012-09-05 18:45:00" />

	   <CreateOrUpdate> ... </CreateOrUpdate>
	   <CreateOrUpdate> ... </CreateOrUpdate>
	   <CreateOrUpdate> ... </CreateOrUpdate>

	   <Delete> ... </Delete>
	   <Delete> ... </Delete>

	</Changes>

#### 5.4.5 Change Events

##### 5.4.5.1 Changes Root Node

- The `Changes` element will include the `clientId` of the request.  This will be
  the same ID that was passed into the `GetChanges` web method call.
- The `Changes` element will include a `commitToken`, which will be some arbitrary
  encoded text string.
- This `commitToken` must be passed verbatim into the next `GetChanges` call as an
  acknowledgment that the current set of changes has been successfully applied
	to the client’s local database, and the next set of changes can be safely
	returned.
- If the client calls `GetChanges`, and there are no changes in the queue, the
  `commitToken` attribute will be missing and the `Changes` element will have no
	child elements.  When the client receives this result, it may stop calling the
	`GetChanges` method and await a `NotifyChangesAvailable` event.

##### 5.4.5.2 CreateOrUpdate Node

- The `CreateOrUpdate` node is sent under two cases:
	- The containing object has been created
	- The containing object has been updated

- One of the following elements will appear as a child of this node (see
  _Appendix A_ for details about these object types):
	- `Offices`
	- `Agents`
	- `Listings`
	- `Developments`
	- `AreaTree`

- With the exception of the `AreaTree`, the identity of the containing object
  will be present as an `id` attribute on the object itself (see _Appendix A_
	for details).
- Since the `AreaTree` is always sent as a complete object within a single
  `GetChanges` call, there is no need to supply an ID.  The client must update
	its area tree to match the new data as supplied in the `AreaTree` object.
- When the client processes this event, it should add the object if it doesn’t
  exist in its local database, or update the object to this new state if it
	already exists.

##### 5.4.5.3 Delete Node

- The `Delete` node is sent when the containing object is to be removed from the
  client’s database.
- One of the following elements will appear as a child of this node:
	- `OfficeRef`
	- `AgentRef`
	- `ListingRef`
	- `DevelopmentRef`
- These Ref elements have no children and a single `id` attribute that refers to
  the unique ID of the corresponding data object type.
- When the client processes this event, it should remove the object from its
  local database if it exists.

##### 5.4.5.4 BeginSnapshot Node

- The `BeginSnapshot` node is sent when a full snapshot of a set of objects is
  to be sent to the client.
- The `type` attribute denotes which object types are included in the snapshot.
  The object types will be a comma-separated list of one or more of the following:
	- `Offices`
	- `Agents`
	- `Listings`
	- `Developments`
	- `AreaTree`
- The `BeginSnapshot` element will be followed by a repeating series of
  `Snapshot` elements, and terminated with an `EndSnapshot` element.  Together
	these three components form a “transaction".
- The client needs to read all Snapshot objects between the Begin and End
  element markers.  On receipt of the `EndSnapshot`, it should compare these
	objects against its own database, adding any missing items, updating any
	existing items, and removing any items in its database that do not appear
	in the set of Snapshot objects just received.  Alternately, the client may
	choose to delete all objects in its database and replace them with the objects
	just received in the snapshot. The effect of performing this update will fully
	synchronise the client’s database with the Sync Store’s database.
- If another `BeginSnapshot` element is received before an `EndSnapshot`, this
  will indicate that the snapshot has been aborted and new snapshot started.  
	The client should discard all snapshot items collected so far and restart the
	collection process.
- **NOTE**: The `BeginSnapshot` and `EndSnapshot` elements may appear across
  multiple calls to `GetChanges` due to the size of the snapshot request!  DO
	NOT expect all the changes to be in the same Changes element.

##### 5.4.5.5 EndSnapshot Node

See the `BeginSnapshot` section above for details on this element.

##### 5.4.5.6 Snapshot Node

- The `Snapshot` element always appears between a `BeginSnapshot` and an `EndSnapshot` element.
- One of the following elements will appear as a child of this node (see _Appendix A_ for details about these object types):
	- `Office`
	- `Agent`
	- `Listing`
	- `Development`
	- `AreaTree`

##### 5.4.5.7 Rollback Node

- The `Rollback` element indicates that the Sync Store has been “wound back" to
  the “to" time.  The Sync Store will resend `CreateOrUpdate` and `Delete`
	events from this “to" time in the past, up until the present time.
- The “to" time will be represented in GMT+0 (NOT local time!).
- This element is sent in response to the client calling the `RequestRollback`
  method.
- See the `RequestRollback` method for more details.

#### 5.4.6 Exceptions on Failure

- The following method-specific exceptions may occur (see Appendix for details):
	- Invalid Commit Token
	- Commit Token Expired
- The following common exceptions may occur (see Appendix for details):
	- Invalid Client ID
	- Invalid Security Token
	- Security Token Expired
	- Internal Error
	- Service Offline 

### 5.5 Notify Changes Available <a id="55-notify-changes-available"></a>

#### 5.5.1 Overview

- This is an OUTGOING event from the Sync Store to an endpoint supplied by the
  client.
- The Sync Store will invoke this web method whenever new events are available
  in the Changes queue, ready to be retrieved by the client.
- The client must complete processing of this call within 1 minute.  If the
  Sync Store does not receive a response within this time period, the
	notification call will be assumed to have failed.
- If this call fails for any reason, the Sync Store will continue to invoke it
  every minute for the next 10 mins, then every 10 mins for the next hour,
	then every hour thereafter.
- **NOTE**: The client should NOT invoke `GetChanges` from within the context of
  this call, since this could cause the notification method to timeout.
- This call will be rate-limited to a max rate of 1 call every 10 seconds.

#### 5.5.2 Web Method Specification

> Name
> : `NotifyChangesAvailable`
>
> HTTP Verb
> : `POST`
>
> | Parameters            | Type            |           |
> |:----------------------|:----------------|:----------|
> | (`securityTokenParams`) | (SecurityToken) | Mandatory |
> | `clientId`              | Int             | Mandatory |
>
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://client-host/client-endpoint`
>
> **Notes**
> : This method is invoked on a CLIENT ENDPOINT and NOT on the SYNC STORE WEB API ENDPOINT.

#### 5.5.3 Parameters

##### 5.5.3.1 clientId

- This parameter will always be present.
- It will match the Client ID as supplied to the client.

##### 5.5.3.2 (securityTokenParams)

- These parameters will always be present.
- The client will need to verify the Token (see the Authentication section for details).

#### 5.5.4 Returns on Success

##### 5.5.4.1 An Example

	<RequestCompleted/>

##### 5.5.4.2 Root Node

The client should simply return an empty `<RequestCompleted>` XML element.

## 6 Web Methods: Enquiries <a id="6-web-methods-enquiries"></a>

### 6.1 Send Enquiry <a id="61-send-enquiry"></a>

#### 6.1.1 Overview

- The client calls this method to send an Enquiry to the Fusion system.
- This call is made by the client on behalf of a human end-user (the “lead").

#### 6.1.2 Web Method Specification

> Name
> : `SendEnquiry`
>
> HTTP Verb
> : `POST`
>
> | Parameters              | Type               |                                                       |
> |:------------------------|:-------------------|:------------------------------------------------------|
> | (`securityTokenParams`) | (SecurityToken)    | Mandatory                                             |
> | `clientId`              | Int                | Mandatory                                             |
> | `name`                  | String             | Mandatory                                             |
> | `email`                 | String             | Mandatory                                             |
> | `message`               | Multi-line String  | Mandatory                                             |
> | `mobile`                | String             | Optional                                              |
> | `subject`               | Single-line String | Optional                                              |
> | `agentIds`              | String             | Optional                                              |
> | `officeId` *            | Int                | * Exactly one of these IDs or String must be supplied |
> | `listingId` *           | Int                | * Exactly one of these IDs or String must be supplied |
> | `fusionRef` *           | String             | * Exactly one of these IDs or String must be supplied |
> | `nomail`                | String             | Optional                                              |
>
> Returns
> : Xml string (see below)
>
> Example
> : `POST` to `http://website/Fusion/IncomingEnquiries/SendEnquiry`

#### 6.1.3 Parameters

##### 6.1.3.1 clientId

- This parameter is mandatory but should only be sent in the querystring (as
	part of securityTokenParams) and not repeated in the body of the request.
- It must match a Client ID as setup in the Fusion system.
- The clientId will be recorded as the “source" of the enquiry by the Fusion
  system.

##### 6.1.3.2 name

- This parameter is mandatory.
- The `name` is the full name of the Lead.

##### 6.1.3.3 email

- This parameter is mandatory.
- The `email` is the email address of the Lead.

##### 6.1.3.4 message

- This parameter is mandatory.
- The `message` is the body of the enquiry as submitted by the Lead.
- The description text will be alphanumeric with no HTML encoding.
- The description text may include `\n` or `\r\n` sequences to represent a
  paragraph break.
- Note that a “blank line" paragraph separator should be represented as a run of
  2 paragraph breaks (e.g. `\n\n` or `\r\n\r\n`)

##### 6.1.3.5 mobile

- This parameter is optional.
- The `mobile` is the mobile number of the lead.
- If no mobile number is supplied by the Lead, this parameter may be omitted.

##### 6.1.3.6 subject

- This parameter is optional.
- The `subject` is an optional single line of text for the enquiry.
- If the subject is omitted, the start of the message body will be used in the
  Fusion system in lieu of the subject.

##### 6.1.3.7 agentIds

- This parameter is optional.
- The `agentIds` parameter is a comma-separated list of one or more agent ids.
- These agent ids are the Agent(s) who are associated with this enquiry and
  should be notified of this enquiry.
- The `agentIds` parameter may be omitted if the enquiry is of a general nature
  and is not related to a specific listing.

##### 6.1.3.8 listingId

- Exactly one of the `listingId` or `officeId` or `fusionRef` attributes must be
  present.
- Either the `listingId` OR the `fusionRef` should be supplied if the enquiry is
  related to a specific listing.

##### 6.1.3.9 fusionRef

- Exactly one of the `listingId` or `officeId` or `fusionRef` attributes must be
  present.
- Either the `listingId` OR the `fusionRef` should be supplied if the enquiry is
	related to a specific listing.

##### 6.1.3.10 officeId

- Exactly one of the `listingId` or `officeId` or `fusionRef` attributes must be
  present.
- The `officeId` should be supplied if the enquiry is of a general nature and
  is not related to a specific listing.

##### 6.1.3.11 Nomail

- The `nomail` should be supplied with a value of `true` if the enquiry email to
  the agent should not be sent (for a specific listing). The parameter is
	optional and defaults to `false`.

#### 6.1.4 Returns on Success

##### 6.1.4.1 An Example

	<EnquirySent/>

##### 6.1.4.2 Root Node

If the enquiry send was successful, an `EnquirySent` element will be returned.

#### 6.1.5 Exceptions on Failure

##### 6.1.5.1 An Example

	<EnquiryError>
		<Exception type="InvalidParameter" paramName="listingId" />
	</EnquiryError>

- The following method-specific exceptions may occur when these parameters are missing.
	- Missing Parameter (name)
	- Missing Parameter (email)
	- Missing Parameter (message)
	- Listing ID or Office ID or fusionRef Missing
- The following method-specific exceptions may occur when the value passed does not match a value in our database, within the given context for the client in question:
	- Invalid Parameter (name)
	- Invalid Parameter (email)
	- Invalid Parameter (message)
	- Listing ID or Office ID or fusionRef Missing
- The following common exceptions may occur (see Appendix for details):
	- Invalid Client ID
	- Invalid Security Token
	- Security Token Expired
	- Internal Error


## 7. APPENDIX A: SYNC OBJECT TYPES <a id="7-appendix-a-sync-object-types"></a>

### 7.1 Agent <a id="71-agent"></a>

#### 7.1.1 An Example

	<Agent id="2"
		 firstName="Jane"
		 surname="Sinclair"
		 role="Manager|Agent|Admin"
		 tel="031 456 5678"
		 cell="084 555 1234"
		 email="jane@remax.co.za"
		 profilePicUrl="http://imageserver/somepath/2/pic-small.jpg"
		 profilePicUrl2="http://imageserver/somepath/2/pic-high-res.jpg"
		 title="Team Leader"
		 team="Uptown Patriots" />

#### 7.1.2 Agent Node

- The `id` is an integer ID representing the Agent.  This ID will be unique
  across all Agents in the Fusion system.
- The `role` attribute can be one of the following values: Agent, Admin or
  Manager.
- The `profilePicUrl` will point to an image in JPG format at some publically
  addressable URL.  The image will be 88 pixels in width and 109 pixels in
	height.  The client must download this image and store it in its own image
	repository – they may NOT expose this URL directly on their site.
- A larger format image is also available via the `profilePicUrl2` attribute,
  which will serve up an image 300px wide by 420px high. Consumers may decide
	which of these images to use.
- If the agent has no picture, `profilePicUrl` and `ProfilePicUrl2` will be an
  empty string.
- The `tel` and `cell` parameters are both mandatory.  Both parameters will be
  formatted with spaces after the 3rd and 6th digits.
- The `title` and `team` attributes are not mandatory.

### 7.2 Office <a id="72-office"></a>

#### 7.2.1 An Example

	<Office id="6"
      agency="Remax"
      branch="Durban North"
      address="123 Smith Street, Craighall 2196"
      tel="011 123 5678"
      fax="011 123 5699"
      email="dnorth@remax.co.za">

	   <Agents>
	      <AgentRef id="123"/>
	      <AgentRef id="432"/>
	      <AgentRef id="562"/>
	   </Agents>

	</Office>

##### 7.2.1.1 Office Node

- All attributes are mandatory, with the exception of `fax` which is optional.
- The `id` is an integer ID representing the Office.  This ID will be unique across all Offices in the Fusion system.
- The `address` attribute will be the physical address.
- The `tel` and `fax` attributes will be formatted with spaces after the 3rd and 6th digits.

##### 7.2.1.2 Agents and AgentRef Nodes

- An AgentRef element will be returned for each member of the office.
- The AgentRef `id` attribute will reference a valid ID of an Agent in the data store.
