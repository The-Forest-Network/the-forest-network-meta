# 0005 - Initial Channels For The Forest Network Launch

**Date:** 2026-08-02

## Context

We carried over the existing channels and spaces from our Forest Network trial when we made a new Matrix server. The initial structure was spaces defined by large geographic regions (Pangaea, Oceania, etc.). Our desire was to have as little fragmentation as possible by concentrating as many channels as possible in the Pangaea space. This was to promote as much unity and connection as possible. Our goal for smaller regions like Oceania, the Americas, etc. would be to only have channels specific to those areas (perhaps around indigenous wildlife, plants, customs, events, etc.). This was to recognise that there is also meaningful conversation and connection to be found for folks living in those bio-regions.

For our initial release of The Forest Network Digital Village we are thinking it would be best just to start with the Pangaea space so as to foster maximum connection. Most likely, adoption will be slow and we feel that having multiple different spaces might make folks feel isolated until such time as our village grows large enough to support multiple spaces. We would like spaces to grow organically and naturally as needed.

Channels in Matrix cannot be deleted in the way that channels in other chat servers or messaging apps can (e.g., by pressing a "delete" button). In order to delete a channel on a Matrix server, all members have to leave and then the channel is deleted eventually (https://its.h-da.io/element-docs/en/rooms/delete/). Furthermore, as a corollary to this decision we will have to update a Synapse config file that automatically adds new members to most of these channels.

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

## Decision

We will remove the following channels:

| Space   | Channel                                    | Visibility | Description                                                                                         |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |

We will keep/add the following channels:

| Space   | Channel                                    | Visibility | Description                                                                                         |
| ------- | ------------------------------------------ | ---------- | --------------------------------------------------------------------------------------------------- |

## Consequences

- For any channels being deleted, all existing members will have to leave that channel
- We will need to update the Synapse config file where new members are automatically added to channels
- We should not onboard new members until this decision record is closed, otherwise we risk difficulty deleting channels (and confusion)