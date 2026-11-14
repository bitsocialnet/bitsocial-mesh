# Bitsocial Mesh

**Bitsocial Mesh is an experimental "Nearby" setting for Bitsocial clients such as 5chan, Seedit and the future Twitter/X alternative.** With it turned on, phones close to each other exchange signed Bitsocial posts over a Bluetooth mesh when there is no internet, and each author's app publishes their own posts to the wider Bitsocial network once it is back online. Longer-range carriers such as LoRa radios come later.

> **Status: idea stage.** This repository reserves the name and collects the design. Nothing here is built. It is not on the current roadmap: P2P chat on Bitsocial itself comes first. It fits [Phase 5 of the Bitsocial master plan](https://bitsocial.net/#master-plan-phase-5), the long tail of niche clients.

## The Thesis

Every Bitsocial publication is already signed by its author's key, and every community is already addressed by its owner's key. Neither depends on the network that carried them. A post that crossed a Bluetooth mesh, a LoRa link or a USB stick can be verified exactly like one that came over libp2p.

So nearby posting needs no new protocol and no new client. It needs a mesh transport and a few shared pieces that existing clients turn on from their settings.

## A Setting, Not a Separate App

| Piece | Where it would live | What it does |
|---|---|---|
| Bluetooth mesh plugin | A Capacitor plugin used by the clients' Android builds | Advertises, scans and relays packets between nearby phones, with hop limits and duplicate suppression. |
| Mesh sync layer | Shared with the clients through bitsocial-react-hooks | Broadcasts the user's signed publications, syncs other people's from nearby phones, verifies them with pkc-js and queues the user's own posts for publishing. |
| Nearby UI | Each client | A settings toggle, a Nearby view and a queued, published or rejected state on the user's own posts. |

Platforms:

- **Android** (Capacitor) is the first target.
- **Desktop** (Electron) may work later. Not yet checked.
- **Web and PWA builds can't take part.** Web Bluetooth can connect out to a device but can't advertise or accept connections, so a browser tab can't be a mesh node.
- **iOS** needs an iOS build of the clients first. 5chan and Seedit don't ship one.

## Why It Needs No Protocol Change

A community node rejects publications timestamped more than 5 minutes from its clock. That stops a relay or a gateway from delivering someone else's post hours later, but it doesn't stop the author: when the author's app is back online it signs each queued post again with a fresh timestamp and publishes it like any other post. Nearby phones see the offline copy; the community gets the new one. See [how-it-could-work.md](how-it-could-work.md#the-5-minute-rule).

The best fit is a profile-based client like the [Phase 3 Twitter/X alternative](https://bitsocial.net/#master-plan-phase-3). A profile is a community that only its owner can post to, so the owner's queued posts need nobody else's approval. In 5chan and Seedit it works too, but the Nearby view is unmoderated until posts reach their board, and the board can still reject them.

## Compared With bitchat

[bitchat](https://github.com/permissionlesstech/bitchat) is a chat app whose phones form a Bluetooth mesh, with Nostr relays as its internet path. Its design is documented in its [whitepaper](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md), and Bitsocial Mesh would borrow its mesh design.

| | bitchat | Bitsocial Mesh |
|---|---|---|
| What nearby people share | Public chat, end-to-end encrypted private messages and, in the iOS app since July 2026, short signed board notices | Posts, replies and votes in Bitsocial communities and profiles |
| How long it lasts | Public chat history syncs for 6 hours. Board notices are at most 512 bytes and last at most 7 days. | Nearby copies are temporary. A post lasts as long as its community keeps it once accepted. |
| Internet path | Nostr relays: location channels and private messages | The author's own client publishes to Bitsocial when back online |
| Moderation | Users block senders | Users block authors in the Nearby view; each community moderates posts that reach it |
| Private messages | Yes | No. Bitsocial has no DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)). |

The iOS and macOS app is public domain (Unlicense). [bitchat-android](https://github.com/permissionlesstech/bitchat-android) is a separate Kotlin codebase under GPL-3.0, so its Bluetooth code could be reused in GPL clients.

## Later: Beyond Bluetooth

LoRa, packet radio, gateways and communities hosted on the mesh itself are later stages. They run into the 5-minute rule directly, because a gateway would be delivering posts the author signed earlier. [Reticulum](https://reticulum.network/) is the leading candidate transport for them. See [how-it-could-work.md](how-it-could-work.md#later-longer-range) and the official [Bitsocial and Reticulum](https://docs.bitsocial.net/comparisons/reticulum/) comparison.

## Documents

- [how-it-could-work.md](how-it-could-work.md): the Nearby setting, publishing on reconnect, privacy, and the later long-range stages
- [ROADMAP.md](ROADMAP.md): the phases from first measurements to an experimental Android release, with a checklist for each
- [open-questions.md](open-questions.md): decisions not yet made, including wire format, unmoderated content, licensing and radio law

## License

[GPL-3.0-or-later](LICENSE). bitchat-android's licence file has been GPL-3.0 since February 2026, while its README still says public domain, and it does not say whether later GPL versions apply. Confirm its terms before reusing its code. Reticulum uses its own [Reticulum License](https://reticulum.network/manual/license.html), which adds use restrictions to MIT-style terms. Read it before bundling Reticulum code into anything here.

## Ecosystem Context

- Bitsocial protocol docs: https://bitsocial.net
- Bitsocial and Reticulum comparison: https://docs.bitsocial.net/comparisons/reticulum/
- Bitsocial and Secure Scuttlebutt comparison (the closest Bitsocial-adjacent offline design): https://docs.bitsocial.net/comparisons/scuttlebutt/
- Protocol implementation: `pkcprotocol/pkc-js`
- Shared client hooks: `bitsocial-react-hooks`
- Clients that could add the setting: 5chan (imageboard), Seedit (forum)
