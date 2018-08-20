---
layout: post
title:  "Updated to Rev 21 - Agent User Titles"
date:   2017-10-13 08:00:00 +0200
---
The [documentation](/FeedStoreAPI/docs) was updated to the latest revision: 21

Non-breaking change: An optional `title` attribute has been added to the list of `<AgentRefs>` for an Office. This title should be preferred to the one on the `Agent` record i cases where the agent appears in more than one office.

# Agent Titles
The list of `AgentRefs` for an office will now contain an optional `title` attribute that contains the Agent title for *THAT* specific office.

The attribute may contain a title or an empty string.

_Please note that the `title` attribute will still be passed on the *Agent* record, but in cases where the agent is in multiple offices, it would be best to override that value with the one for the office concerned._

Listing Enquiries


_example_

```
<Office id="244" agency="Epic Realty" branch="Pro Demo" address="Ravenscraig Road,&#xD;&#xA;Woodstock" tel="021 447 4000" fax="" email="mached@privateproperty.co.za">
<Agents>
  <AgentRef id="832810" title="Administration" />
  <AgentRef id="3464" title="Principal" />
  <AgentRef id="19308" title="" />
  <AgentRef id="12875" title="Agent" />
</Agents>
```