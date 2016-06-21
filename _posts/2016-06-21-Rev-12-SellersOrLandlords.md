---
layout: post
title:  "Updated to Rev 12 - Sellers And Landlords"
date:   2016-06-21 12:35:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 12

Non-breaking addition: Added Seller and Landlord details to the feed.

- We've added a new <SellersAndLandlords> node to the feed that will house any number of child elements relating to seller or landlord information.
Please note that in cases where the listingType=Sale, then the SellerOrLandlord data will assume Seller, and in cases where listingType=Rent, then it will assume Landlord.
