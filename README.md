# Bitsocial Mesh

**Bitsocial Mesh is a plan to keep Bitsocial communities working when the internet is gone**, whether a government switched it off or a disaster took it down. People in the same place keep reading, posting, replying and moderating in apps such as 5chan and Seedit. When any link to the outside returns, what they wrote reaches the rest of the network.

> **Status: idea stage.** This repository reserves the name and collects the design. Nothing here is built or tested. See [ROADMAP.md](ROADMAP.md). It fits [Phase 5 of the Bitsocial master plan](https://bitsocial.net/#master-plan-phase-5).

## The Problem

Offline mesh chat already exists. [bitchat](https://github.com/permissionlesstech/bitchat) lets nearby phones talk over Bluetooth with no internet. What a mesh chat does not have is communities: threads that persist, rules, moderators and a defence against spam. In a blackout, a crowd sharing one unmoderated channel is easy to flood and hard to trust.

Bitsocial has communities. What it lacks is a way to run them without the internet.

## Why Bitsocial Can't Copy bitchat

A bitchat message is valid as soon as it is signed, so phones can pass it along to each other. A Bitsocial post has to reach its community's host node, which accepts it, gives it its permanent identifier, runs the anti-spam challenge and applies moderation. If nobody can reach the host, nothing is accepted, and nobody can reply or vote.

So the question is not how phones talk to each other. It is how people reach a host, and how a host reaches the world.

## The Plan

| Stage | What it is | What it needs |
|---|---|---|
| **1. The community box** | A laptop or a small computer hosts the community and opens a Wi-Fi hotspot. People nearby join the hotspot and open the app in a browser. Posts are accepted, threaded, challenged and moderated as usual. | No protocol change. Whether today's software already works this way is untested. |
| **2. Delayed delivery** | A post signed inside the blackout is carried out by any device that later reaches the internet, and its community still accepts it. This is the way out of a shutdown that needs no hardware. | One opt-in protocol change. Today a host rejects a post timestamped more than 5 minutes from its clock. |
| **3. Linking boxes** | Boxes across a city reach each other, and a box with internet relays for the others, over LoRa radios. | Radio hardware, run by a few volunteers. Not something each user carries. |
| **4. Phone to phone** | Phones exchange posts directly over Bluetooth, as bitchat does. | A native Bluetooth mesh in the Android apps. The most work for the shortest range, so it comes last and may be dropped. |

Details are in [how-it-could-work.md](how-it-could-work.md).

## What Runs Where

| Piece | Where it would live | What it does |
|---|---|---|
| Community box | A desktop app on a laptop, or an image for a small computer such as a Raspberry Pi | Runs the community's node, opens a hotspot and serves the client to browsers on it. |
| Delayed delivery | pkc-js, and the clients through bitsocial-react-hooks | Lets a community opt in to older posts, and lets any client carry other people's posts out. |
| Radio link | Beside a box's node | Carries posts and community updates between boxes. |
| Bluetooth mesh | A Capacitor plugin in the clients' Android builds | Phone-to-phone exchange, if Stage 4 is built. |

## Limits

- **Range.** A hotspot covers a building, a square or a camp. Beyond that it takes more boxes, radio links or people carrying data.
- **Someone has to run the box.** The community stops if the box is switched off, seized or out of power, and the person running it can be located by its radio signal.
- **A box has to be prepared before the blackout.** Nobody can download it afterwards. People who only join one need nothing but a browser, if serving the app from the box works.
- **Privacy.** Joining a hotspot shows the box that a device is there, and every post carries its author's public key.

## Compared With bitchat

bitchat's design is documented in its [whitepaper](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md).

| | bitchat | Bitsocial Mesh |
|---|---|---|
| What people share | Public chat, end-to-end encrypted private messages and, in the iOS app since July 2026, short signed board notices | Posts, replies and votes in Bitsocial communities and profiles |
| Needs a host to be reachable | No. A message is valid as soon as it is signed. | Yes, which is why the host has to be nearby. |
| Needs an app installed | Yes | A browser, for people joining a box |
| How long it lasts | Public chat history syncs for 6 hours. Board notices are at most 512 bytes and last at most 7 days. | As long as the community keeps it. |
| Internet path | Nostr relays: location channels and private messages | The community is an ordinary Bitsocial community once it has a link out |
| Moderation and spam | Users block senders | The community's own rules, moderators and challenges |
| Private messages | Yes | No. Bitsocial has no DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)). |

## Later: A Map

Sites such as [bitchatexplorer.com](https://bitchatexplorer.com/) and [bitmap.lat](https://bitmap.lat/) draw bitchat's location channels on a world map. They read those from Nostr relays over the internet, not from the Bluetooth mesh. A Bitsocial map of communities by coarse location is possible the same way, as an opt-in feature that only works online.

## Documents

- [how-it-could-work.md](how-it-could-work.md): the four stages, the 5-minute rule, measured sizes and privacy
- [ROADMAP.md](ROADMAP.md): the phases, with a checklist and a pass-or-stop check for each
- [open-questions.md](open-questions.md): decisions not yet made

## License

[GPL-3.0-or-later](LICENSE). Two notes for later stages. [bitchat-android](https://github.com/permissionlesstech/bitchat-android)'s licence file has been GPL-3.0 since February 2026, while its README still says public domain, and it does not say whether later GPL versions apply. [Reticulum](https://reticulum.network/) uses its own [Reticulum License](https://reticulum.network/manual/license.html), which adds use restrictions to MIT-style terms. Confirm both before reusing or bundling their code.

## Ecosystem Context

- Bitsocial protocol docs: https://bitsocial.net
- Bitsocial and Reticulum comparison: https://docs.bitsocial.net/comparisons/reticulum/
- Bitsocial and Secure Scuttlebutt comparison (the closest Bitsocial-adjacent offline design): https://docs.bitsocial.net/comparisons/scuttlebutt/
- Protocol implementation: `pkcprotocol/pkc-js`
- Shared client hooks: `bitsocial-react-hooks`
- Clients: 5chan (imageboard), Seedit (forum)
