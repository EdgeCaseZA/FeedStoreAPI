---
layout: post
title:  "Updated to Rev 19 - Holiday Rentals"
date:   2017-04-19 08:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 19

Non-breaking changes: New functionality in Fusion now allows users to flag a Rental listing as a HolidayRental and as a result we've made the following changes to the feed.

* New attribute inside `RentDetails` element: `isHolidayRental="true"` denotes if holiday rental. Absence indicates it is not.
* New `HolidayRentalDetails` element with some, or all of the following attributes.
`highSeasonRate`, `highSeasonStartDate`, `highSeasonEndDate`, `midSeasonRate`, `midSeasonStartDate`, `midSeasonEndDate`, `lowSeasonRate`, `lowSeasonStartDate`, `lowSeasonEndDate`, `offSeasonRate`, `offSeasonStartDate`, `offSeasonEndDate`, `corporateRate`, `priceSuffix`
* New secondary property feature: `NumPeopleSleeps` can be used to denote the number of people the property can accommodate.

_example_
```<RentDetails rentalState="ToRent" occupationDate="2014-09-30" rentalPrice="4800" deposit="4800" waterIncluded="false" electricityIncluded="false" isHolidayRental="true" />```

```<HolidayRentalDetails highSeasonRate="800" highSeasonStartDate="2017-04-01" highSeasonEndDate="2017-04-30" midSeasonRate="0" lowSeasonRate="0" offSeasonRate="0" corporateRate="200" />```

```<SecondaryFeatures arePetsAllowed="true" hasAccessGate="true" hasKitchen="true" hasPaving="true" numPeopleSleeps="4" />```