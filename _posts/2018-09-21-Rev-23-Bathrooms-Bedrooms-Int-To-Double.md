---
layout: post
title:  "Updated to Rev 23 - Changes to Bedrooms and Bathrooms Number Type"
date:   2018-09-21 13:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 23

Potentially a breaking change: The underlying data type for numBedrooms and numBathrooms in the MainFeatures node was changed from an int to a double, and as a result you may recieve floating point numbers going forward.

Please urgently update your integrations to make sure you handle accordingly.

```
<MainFeatures numBedrooms="2.5" numBathrooms="1.5" numGarages="2" numCoveredParkings="3" numOpenParkings="2" landArea="0" landAreaUnits="sqm" floorArea="0" floorAreaUnits="sqm" />
```

