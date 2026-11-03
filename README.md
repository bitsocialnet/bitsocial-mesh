# Bitsocial Mesh

**Bitsocial Mesh is a future Bitsocial client and gateway for networks without the internet.** People nearby keep reading and posting to Bitsocial communities over LoRa radios, packet radio, Wi-Fi or Bluetooth while disconnected, and a gateway syncs their posts with the wider Bitsocial network whenever it can reach the internet.

> **Status: idea stage.** This repository reserves the name and collects the design. Nothing here is built. It is not on the current roadmap: P2P chat on Bitsocial itself comes first. It fits [Phase 5 of the Bitsocial master plan](https://bitsocial.net/#master-plan-phase-5), the long tail of niche clients.

## The Thesis

Every Bitsocial publication is already signed by its author's key, and every community is already addressed by its owner's key. Neither depends on the network that carried them. A post that crossed a LoRa mesh, a USB stick or a ham radio repeater can be verified exactly like one that came over libp2p.

So an off-grid Bitsocial does not need a new protocol. It needs:

| Piece | What it does |
|---|---|
| A mesh transport | Moves signed publications between nearby devices. [Reticulum](https://reticulum.network/) is the leading candidate. |
| A mesh client | Reads and writes Bitsocial communities from what the mesh has, and queues publications for later. |
| A gateway | A node with both a mesh interface and internet access. It pulls selected communities into the mesh and pushes queued publications to the communities' own nodes. |
| Optionally, a mesh-hosted community | A community whose node runs on the mesh itself (a Raspberry Pi on a hill), so the mesh has a community that doesn't depend on the internet at all. |

This is the path the official [Bitsocial and Reticulum](https://docs.bitsocial.net/comparisons/reticulum/) comparison describes: a client that works over a local mesh while disconnected and syncs when a peer or gateway with internet access is reachable, built as a new client and bridge rather than a change to the protocol.

## Why Reticulum

Reticulum is a networking stack, not a social protocol. It sits roughly where libp2p sits under Bitsocial, so the two complement each other:

- It runs over almost any carrier, down to links of a few bits per second with a 500-byte MTU.
- It needs no IP addresses, DNS, certificate authorities or accounts.
- LXMF, its messaging layer, already does store-and-forward through propagation nodes, which is the delivery model an off-grid client needs.
- It defines no communities, posts or moderation. That is the part Bitsocial already has.

Reticulum is the working assumption, not a decision. See [open-questions.md](open-questions.md#transport).

## Documents

- [how-it-could-work.md](how-it-could-work.md): mesh client, gateway, posting and syncing
- [open-questions.md](open-questions.md): decisions not yet made, including bandwidth, challenges, licensing and radio law

## License

[GPL-3.0-or-later](LICENSE). Reticulum uses its own [Reticulum License](https://reticulum.network/manual/license.html), which adds use restrictions to MIT-style terms. Read it before bundling Reticulum code into anything here.

## Ecosystem Context

- Bitsocial protocol docs: https://bitsocial.net
- Bitsocial and Reticulum comparison: https://docs.bitsocial.net/comparisons/reticulum/
- Bitsocial and Secure Scuttlebutt comparison (the closest Bitsocial-adjacent offline design): https://docs.bitsocial.net/comparisons/scuttlebutt/
- Protocol implementation: `pkcprotocol/pkc-js`
- Sibling clients: 5chan (imageboard), Seedit (forum)
