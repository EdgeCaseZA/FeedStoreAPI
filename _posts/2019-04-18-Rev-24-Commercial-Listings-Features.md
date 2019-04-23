---
layout: post
title:  "Updated to Rev 24 - New Commercial Listing Features"
date:   2019-04-18 13:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) has been updated to the latest revision: 24

Non-breaking change: Additional listing information pertaining to Commercial Listings (Sales & Rentals) has been added to the XML snapshot where applicable.

*COMING IN THE WEEK OF 22 - 26 APRIL*

The following additonal details is now available across 2 x new elements:
*General Information*
- Lease Type
- Building Grade
- Power Availability
- Gross Rental Per Month
- Net Rental Per Month
- Annual Rental Escalation
- Zoning
- Under Cover Parking Bay Cost Per Month
- Open Parking Bay Cost Per Month
*Specific Features*
- Number of Dock Levellers
- Height of Dock Levellers
- Number of Roller Shutter Doors
- Height of Roller Shutter Doors
- Number of BoardRooms
- Boardrooms Description
- Gross Lettable Area (GLA)
- Number of Floors
- Retail Space
- Mezzanine Floor Size
- Is Standalone Building
- Has Backup Water
- Is Green Building
- Is Multi Tenanted

1. CommercialListingDetails element (optional)
   This element is optional and will only appear in the XML package if any of it's attributes have values. In the event that the element has appeared in a previous snapshot, but not in subsequent snapshots, then it should be understood that those attributes are no longer applicable on the listing and can be cleared from your local store.

```
<CommercialListingDetails leaseType="DoubleNetLease" buildingGrade="AAA" powerAvailability="ThreePhase" grossRentalPerMonth="25000" netRentalPerMonth="20000" escalation="8" zoning="Mixed Use" underCoverParkingBayCostPerMonth="250" openParkingBayCostPerMonth="100" />
```



1. CommercialFeatures element (optional)
This element is optional and will only appear in the XML package if any of it's attributes have values. In the event that the element has appeared in a previous snapshot, but not in subsequent snapshots, then it should be understood that those attributes are no longer applicable on the listing and can be cleared from your local store.

```
<CommercialFeatures numDockLevellers="4" heightOfDockLevellers="3.60" numRollerShutterDoors="2" heightOfRollerShutterDoors="2.80" heightOfRoof="4.00" numBoardrooms="2" boardRoomsDescription="Luxurious, fully kitted out" grossLettableArea="1500" numFloors="5" retailSpace="1080.00" mezzanineFloor="500.00" IsStandaloneBuilding="true" HasBackupWater="true" IsGreenBuilding="true" IsMultiTenanted="true" />
```

Please see the [documentation](/FeedStoreAPI/docs) for more specific information around each feature.

