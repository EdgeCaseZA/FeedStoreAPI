---
layout: post
title:  "Important note about listing feedRef attribute"
date:   2018-08-20 17:00:00 +0200
---
## Important Notification
Please be aware that the `feedRef` attribute should *never* be used as the primary key in the consumer's local database.

The `feedRef` is a _web-friendly_ reference number and is not gauranteed to be unique at a system-wide level.

The listing `id` attribute will *always* be unique and should be used as the unique identifier when referencing any listing that has come down the feed from Fusion.

_The API documentation has been updated to reflect the above_

Section 7.4.1.1 - Listing Node

```The `fusionRef` is an combination of 4 alphanumeric characters following by a hyphen and then 4 numbers. ABCD-0001. It should NOT be considered unique. Use the `id` field to uniquely identify the listing.

The agencyRef could be an empty string, or any combination of alphanumeric and symbolic characters. It should NOT be considered unique. Use the `id` field to uniquely identify the listing.```