---
layout: post
title:  "Updated to Rev 14 - Always send full address"
date:   2016-10-05 09:35:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 14

Non-breaking addition: Added functionality to always send the full listing address (if the Client and Agency flags to allow it is set) 

- The Fusion back office feed-client configuration allows for an indicator `AlwaysSendFullListingAddress` to be set per Client per Agency. 
If this indicator is set to `true`, the Address Node's three address attributes and the two scheme attributes will always be present.
It is up to the client then to ensure that the address is not shown if `addressHidden` is `true`. See [7.4.1.5 Address Node](#7415-addressNode) for details.
