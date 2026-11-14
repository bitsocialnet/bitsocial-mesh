# Bitsocial Mesh

**Bitsocial Mesh is a plan to keep Bitsocial communities working when the internet is gone**, whether a government switched it off or a disaster took it down. People in the same place keep reading, posting, replying and moderating in apps such as 5chan and Seedit, over a mesh of nearby devices. When any link to the outside returns, what they wrote reaches the rest of the network.

> **Status: idea stage.** This repository reserves the name and collects the design. Nothing here is built, and the two capabilities that matter most need protocol work in pkc-js that has not been proposed yet. See [ROADMAP.md](ROADMAP.md). It fits [Phase 5 of the Bitsocial master plan](https://bitsocial.net/#master-plan-phase-5).

## The Problem

Offline mesh chat already exists. [bitchat](https://github.com/permissionlesstech/bitchat) lets nearby phones talk over Bluetooth with no internet. What a mesh chat does not have is communities: threads that persist, rules, moderators and a defence against spam. In a blackout, a crowd sharing one unmoderated channel is easy to flood and hard to trust.

Bitsocial has communities. What it lacks is a way to run them without the internet.

## What Has to Be True

A Bitsocial community lives where its owner's node can be reached. That node accepts each post, gives it its permanent identifier, runs the anti-spam challenge and applies moderation. If nobody can reach it, nothing is accepted, and nobody can reply or vote. So a blackout needs two things that do not exist today:

| Capability | What it means | What it needs |
|---|---|---|
| **A community on the mesh** | A laptop, desktop or small computer inside the affected area hosts the community, and nearby phones reach it over the mesh. Posts are accepted, threaded, challenged and moderated as usual. | A mesh link between clients and the host node. Hosting needs Node.js today, so the host can't be a phone. |
| **Delayed delivery** | A post signed inside the blackout is carried out by any device that later reaches the internet, and its community still accepts it. | A protocol change. Today a community rejects anything timestamped more than 5 minutes from its clock. |

A third, smaller piece needs no protocol change: a **Nearby feed**, where phones exchange signed posts directly and each author's app publishes their own posts once it is back online. It is a useful first step but it is not the goal, because nothing in it is accepted or moderated while the internet is down.

Details are in [how-it-could-work.md](how-it-could-work.md).

## A Setting, Not a Separate App

Everything is meant to live inside the existing clients, turned on from their settings.

| Piece | Where it would live | What it does |
|---|---|---|
| Bluetooth mesh plugin | A Capacitor plugin in the clients' Android builds | Advertises, scans and relays packets between nearby phones. |
| Mesh sync layer | bitsocial-react-hooks, shared by the clients | Sends and receives publications over the mesh, verifies them with pkc-js, queues the user's own posts. |
| Mesh bridge | Beside a community's node, on a desktop app or a small computer | Lets a hosted community take posts from the mesh and send accepted posts back. |
| Mesh UI | Each client | A settings toggle, a view of what is nearby, and the state of each of the user's posts. |

Platforms:

- **Android** (Capacitor) is the first target for phones.
- **Desktop** (Electron) is the natural host for a community on the mesh. Whether it can join a Bluetooth mesh is not checked.
- **Web and PWA builds can't take part.** Web Bluetooth can connect out to a device but can't advertise or accept connections.
- **iOS** needs an iOS build of the clients first. 5chan and Seedit don't ship one.

## Limits

- **Range.** Bluetooth reaches tens of metres per hop. A mesh of phones covers a protest, a campus or a neighbourhood. Anything larger needs LoRa radios or people carrying data.
- **Reliability.** Phone makers restrict background Bluetooth, each in their own way. The realistic target is that it works well on Android while the app is open or recently used.
- **It only helps people who already have the app.** Nobody can install it after the internet is gone, unless the app can be passed from phone to phone.
- **Privacy.** A phone on the mesh advertises that it is there, and every post carries its author's public key.

## Compared With bitchat

bitchat's design is documented in its [whitepaper](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md). Bitsocial Mesh would borrow its mesh design and possibly its packet format.

| | bitchat | Bitsocial Mesh |
|---|---|---|
| What nearby people share | Public chat, end-to-end encrypted private messages and, in the iOS app since July 2026, short signed board notices | Posts, replies and votes in Bitsocial communities and profiles |
| Needs a server to be reachable | No. A message is valid as soon as it is signed. | Yes, the community's host, which is why the host has to be on the mesh. |
| How long it lasts | Public chat history syncs for 6 hours. Board notices are at most 512 bytes and last at most 7 days. | As long as the community keeps it. |
| Internet path | Nostr relays: location channels and private messages | The community is an ordinary Bitsocial community once it has a link out |
| Moderation and spam | Users block senders | The community's own rules, moderators and challenges |
| Private messages | Yes | No. Bitsocial has no DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)). |

The iOS and macOS app is public domain (Unlicense). [bitchat-android](https://github.com/permissionlesstech/bitchat-android) is a separate Kotlin codebase under GPL-3.0, so its Bluetooth code could be reused in GPL clients.

## Later

- **Longer range.** LoRa, packet radio and gateways. [Reticulum](https://reticulum.network/) is the leading candidate transport. See the official [Bitsocial and Reticulum](https://docs.bitsocial.net/comparisons/reticulum/) comparison.
- **A map.** Sites such as [bitchatexplorer.com](https://bitchatexplorer.com/) and [bitmap.lat](https://bitmap.lat/) draw bitchat's location channels on a world map. They read those from Nostr relays over the internet, not from the Bluetooth mesh. A Bitsocial map of communities by coarse location is possible the same way, as an opt-in feature that only works online.

## Documents

- [how-it-could-work.md](how-it-could-work.md): the two capabilities, the Nearby feed, measured sizes, privacy and longer range
- [ROADMAP.md](ROADMAP.md): the phases, with a checklist and a pass-or-stop check for each
- [open-questions.md](open-questions.md): decisions not yet made

## License

[GPL-3.0-or-later](LICENSE). bitchat-android's licence file has been GPL-3.0 since February 2026, while its README still says public domain, and it does not say whether later GPL versions apply. Confirm its terms before reusing its code. Reticulum uses its own [Reticulum License](https://reticulum.network/manual/license.html), which adds use restrictions to MIT-style terms. Read it before bundling Reticulum code into anything here.

## Ecosystem Context

- Bitsocial protocol docs: https://bitsocial.net
- Bitsocial and Reticulum comparison: https://docs.bitsocial.net/comparisons/reticulum/
- Bitsocial and Secure Scuttlebutt comparison (the closest Bitsocial-adjacent offline design): https://docs.bitsocial.net/comparisons/scuttlebutt/
- Protocol implementation: `pkcprotocol/pkc-js`
- Shared client hooks: `bitsocial-react-hooks`
- Clients that could add the setting: 5chan (imageboard), Seedit (forum)
