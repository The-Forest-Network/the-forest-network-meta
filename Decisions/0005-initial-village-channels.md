# 0005 - Initial Channels For The Forest Network Launch

**Date:** 2026-08-02

## Context

We carried over the existing channels and spaces from our Forest Network trial when we made a new Matrix server. The initial structure was architected as spaces named after large geographic regions (Pangaea, Oceania, etc.) which tied into the concept of a global village. Our desire now is to have as little fragmentation as possible by concentrating as many channels as possible in the Pangaea space. This is in order to promote as much unity and connection as possible. Our goal for smaller regions like Oceania, the Americas, etc. would be to only have channels specific to those areas (perhaps around indigenous wildlife, plants, customs, events, etc.). This would be to recognise that there is also meaningful conversation and connection to be found for folks living in those bio-regions.

For our initial release of The Forest Network Digital Village we are thinking it would be best just to start with the Pangaea space so as to foster maximum connection. Most likely, adoption will be slow and we feel that having multiple different spaces might make folks feel isolated until such time as our village grows large enough to support multiple spaces. We would like spaces to grow organically and naturally as needed.

Channels in Matrix cannot be deleted in the way that channels in other chat servers or messaging apps can (e.g., by pressing a "delete" button). In order to delete a channel on a Matrix server, all members have to leave (or be removed by an admin) and then the channel is deleted eventually (https://its.h-da.io/element-docs/en/rooms/delete/). Furthermore, as a corollary to this decision we will have to update a Synapse config file that automatically adds new members to most of these channels.

This decision is not meant to be updated for any future channels (unless a big rearchitecture takes place and a decision around that is needed); just for the initial launch.

### Current Spaces

Current spaces we have are:

| Space   | Visibility | Description                                                        |
| ------- | ---------- | ------------------------------------------------------------------ |
| Pangaea | Public     | Space for channels that concern all Villagers and the entire earth |
| Oceania | Public     | Space for channels relevant to Villagers in the Oceania region     |

### Current Channels

Current channels we have are:

| Space   | Channel                                    | Visibility | Description                                                                                         |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Pangaea | Pangaea/Technical                          | Public     | Top-level channel for users to report technical issues or to ask technical questions about the apps |
| Pangaea | Pangaea/Poetry                             | Public     | Top-level channel to post poetry                                                                    |
| Pangaea | Pangaea/Mycorrhiza Musings                 | Public     | Top-level channel to post soul and nature musings                                                   |
| Pangaea | 🌏 Pangaea/Welcome to the Forest Network 🪾 | Public     | Top-level channel to ask product questions about the apps and offer feedback                        |
| Pangaea | 💭🌙 Pangaea/Dreams of the Earth            | Public     | Top-level channel to talk about what people and the earth are dreaming about                        |
| Pangaea | Pangaea/Trailhead                          | Public     | Top-level channel for new villagers to say hi and introduce their region and indigenous lands       |
| Pangaea | 📣 Pangaea/Council                         | Private    | Top-level channel for the Village Council                                                            |
| Pangaea | Pangaea/Tech                               | Public     | Duplicate channel of Pangaea/Technical                                                              |
| Oceania | Oceania/Trailhead                          | Public     | Channel for people from Oceania to introduce themselves (superseded by Pangaea/Trailhead)           |
| Oceania | Oceania/Events in the Village              | Public     | Channel for people in Oceania to talk about events in their area                                    |
| Oceania | Oceania/The Land                           | Public     | Channel for people in Oceania to talk about their region: birds, plants, ecosystem, etc.            |

---

## Decision

We will remove the following channels:

| Space   | Channel                                    | Visibility | Reason                                                                                              |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Oceania | Oceania/Trailhead                          | Public     | We have Pangaea/Trailhead, going with highest-level channel possible                                |
| Pangaea | Pangaea/Technical                          | Public     | We have Pangaea/Tech; repurpose this one to talk about general Forest Network tech                  |
| Oceania | Oceania/The Land                           | Public     | Not time to differentiate this yet; Pangaea should have this equivalent if we have it at all        |
| Oceania | Oceania/Events in the Village              | Public     | Going to add this at the Pangaea level at least initially                                           |
| Pangaea | Pangaea/Poetry                             | Public     | Need something broader, maybe it should be Mycorrhiza Musings?                                      |

We will add the following channels:

| Space   | Channel                                    | Visibility | Reason                                                                                              |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Pangaea | Pangaea/Events In The Village              | Public     | As we're just starting out, this should be a top-level channel until we get enough Villagers        |
| Pangaea | Pangaea/Village Drop-Box                   | Public     | This is for any and all feedback about The Forest Network apps, direction, etc.                     |
| Pangaea | Pangaea/The Land                           | Public     | Could be a good place for people to post pictures and identify plants, animals, etc.                |

We will keep the following existing channels:

| Space   | Channel                                    | Visibility | Reason                                                                                              |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Pangaea | 💭🌙 Pangaea/Dreams of the Earth            | Public     | Dreams are so important that we should have a dedicated channel for them                            |
| Pangaea | Pangaea/Trailhead                          | Public     | We need an initial channel for people to introduce themselves                                       |
| Pangaea | 📣 Pangaea/Council                         | Private    | We need something like this and it's useful                                                          |
| Pangaea | Pangaea/Mycorrhiza Musings                 | Public     | Top-level channel to post soul and nature musings                                                   |

Final channel list:

| Space   | Channel                                    | Visibility | Description                                                                                         |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |
| Pangaea | Pangaea/Events In The Village              | Public     | As we're just starting out, this should be a top-level channel until we get enough Villagers        |
| Pangaea | Pangaea/Village Drop-Box                   | Public     | This is for any and all feedback about The Forest Network apps, direction, etc.                     |
| Pangaea | 💭🌙 Pangaea/Dreams of the Earth            | Public     | Dreams are so important that we should have a dedicated channel for them                            |
| Pangaea | Pangaea/Trailhead                          | Public     | We need an initial channel for people to introduce themselves                                       |
| Pangaea | 📣 Pangaea/Council                         | Private    | We need something like this and it's useful                                                          |
| Pangaea | Pangaea/Mycorrhiza Musings                 | Public     | Valuable channel to have                                                                            |
| Pangaea | Pangaea/The Land                           | Public     | Could be a good place for people to post pictures and identify plants, animals, etc.                |

---

## Consequences

- For any channels being deleted, all existing members will have to leave that channel
- We will need to update the Synapse config file where new members are automatically added to channels
- We should not onboard new members until this decision record is closed, otherwise we risk difficulty deleting channels (and confusion)