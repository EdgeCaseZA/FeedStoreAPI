---
layout: post
title:  "Update to Rev 18 imminent - Include All Office Ids with Agent object"
date:   2017-03-23 15:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) will soon be updated to the latest revision: 18

_Non-breaking change_
- As an enhancement to the `Agent` object, we've included a new parameter titled `allOfficeIds` which will contain a comma-delimited list of officeIds that the agent is connected to.
_example_
```<Agent id="5807" firstName="Joe" surname="Smith" role="Admin" tel="021 701 9854" cell="083 417 9854" email="joesmith@somewhere.co.za" profilePicUrl="" profilePicUrl2="" title="Principal" team="A Team|Test Team" PrivySealAgentAliasSlug="" allOfficeIds="127,244,801010,801011"  />```
- This represents an agent-office connection and is independent of the <Agents><AgentRef/></Agents> structure which adheres to the `show on website` flag found in Fusion.
- Do not expect that an officeId value in the allOfficeIds attribute points to an exisitng office as it may not yet have been sent downstream - `<Agent />` objects are created before the `<Office />` objects in the feed.

**Important**

_This update only applies to any new `CreateOrUpdate` events that will be applied to Agents. Agent records will need to be published out manually from Fusion in order for this change to reflect via the feed._
