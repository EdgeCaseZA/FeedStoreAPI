---
layout: post
title:  "Updated to Rev 20 - Duplicate Detection on SendEnquiries Endpoint"
date:   2017-06-21 21:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 20

Potentially-breaking change: New DuplicateEnquiry exception added as a Specific Exception Type to the SendEnquiry endpoint and should be expected as a response in situations where duplicate enquiries are posted to the endpoint.

# Duplicate Detection
We've tightened up the logic around our duplicate enquiry detection in order to prevent duplicates finding their way into the system and clogging up the user's new enquiries inbox.

If the following parameters equal the same values of an enquiry already in the database, we will reject the enquiry and throw an EnquiryException of type DuplicateEnquiry

Listing Enquiries

`officeId` *and* `message` *and* `name` *and* `listingId`

General Office Enquiries

`officeId` *and* `message` *and* `name`

_example_

```<EnquiryError>
	<Exception type="DuplicateEnquiry" description="Params match and existing enquiry (officeID, message, name, listingId)." message="Duplicate enquiry detected - please don't spam." />
</EnquiryError>```