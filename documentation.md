---
layout: page
title: Documentation
permalink: /docs/
---

<small>Revision 11 -- Updated 30 July 2015</small>

## Introduction

### 1.1 Overview

This document describes the API for accessing the Fusion Sync Store.

### 1.2 Architectural Overview

The Fusion Sync Store exposes an HTTP endpoint.

Clients request information via that endpoint using a standard request-response web method pattern, as detailed within this document.

Every client is linked to a collection of Fusion Offices.

The client calls the Sync Store API to incrementally retrieve create, update and delete change events for all offices, agents, developments, listings and suburbs.

## 1.3 API Versioning

### 1.3.1 Current Version

* The API is currently on a **major version** of **v1**.

### 1.3.2 API Evolution - Non Breaking Changes

* Additional XML attributes and elements may be _**added**_ to the existing data structures in the future _**without**_ changing the protocol version number.  i.e. your client code must be prepared to deal with new attributes and Elements outside those listed in this document.  However, _**existing**_ attributes and elements will _**not**_ change.

* Additional web methods may be added over time to the same protocol version number.

* Additional _**optional**_ parameters may be added to existing web methods over time to the same protocol version number.

* Additional exception types may be added over time to the same protocol version number.

### 1.3.3 API Evolution - Breaking Changes

* If there is requirement to change any existing url parameter or change any attribute or element in a web method call or XML return structure, a new Protocol Version number will be created, and the previous version of the protocol will be maintained for backwards compatibility purposes for a period of time.

* Should an old protocol be deprecated, all clients of that version will be notified ahead of time and an alternative protocol provided.  The client will be allocated a period of time to transition to the new protocol, before the old version is discontinued.
