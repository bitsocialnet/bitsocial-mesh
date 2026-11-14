# How It Could Work

A sketch, not a spec. It describes the Nearby setting first, then the later long-range stages, and the one protocol rule that shapes both.

## What Already Works Offline

Bitsocial data is self-verifying:

- Every publication (post, reply, vote, edit) is signed by its author's Ed25519 key.
- Every community is addressed by its owner's public key, and its state is signed by that key.

A phone can check a post or a community snapshot it received over Bluetooth or LoRa exactly as it would one received over libp2p. Carrying Bitsocial data across a mesh needs no new trust model.

## The 5-Minute Rule

A community node rejects a publication whose timestamp is more than 5 minutes from its own clock, and drops challenge messages outside the same window. See `ERR_PUBLICATION_TIMESTAMP_IS_NOT_IN_PROPER_RANGE` in [`publication-validation.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/runtime/node/community/local-community/publication-validation.ts#L164) and the pubsub timestamp checks in [`signatures.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/signer/signatures.ts#L1027).

What that allows:

| Who publishes, and when | Accepted with today's protocol? |
|---|---|
| The author's own app, once it is back online, signing at that moment | Yes. It is an ordinary publication. |
| The author's app over a live mesh path to an internet gateway, with the round trip well under 5 minutes | Yes. |
| A relay or gateway delivering a post the author signed earlier | No, once it is more than 5 minutes old. Needs a mesh-hosted community or a protocol change (see [Later](#later-longer-range)). |

The Nearby setting uses the first row. The copy nearby phones saw and the copy the community accepts are therefore different publications, with different timestamps and signatures.

## Stage 1: The Nearby Setting

### Turning It On

An experimental setting, off by default, in the Android builds of 5chan, Seedit and later clients. While it is on, the app runs a foreground service so it keeps advertising and scanning with the screen off. Battery cost and how aggressively phone makers kill background services are unmeasured; both need testing on real devices.

### The Mesh

Every phone is both a Bluetooth central and peripheral and relays packets for others, as in [bitchat](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md):

- a hop limit on every packet, so floods die out,
- duplicate suppression by a hash of each packet,
- a short random delay before relaying, so duplicates are dropped before they are sent,
- fragmentation for publications larger than a Bluetooth packet,
- periodic sync between neighbours of what each phone holds, so a phone that walks from one group to another carries posts with it.

How long a phone keeps and re-shares other people's publications is an open question.

### What Goes Over the Air

- **Publications the user writes while Nearby is on:** posts and replies, signed by the author's key as usual.
- **Votes**, which only matter on comments the community has already accepted, since a vote names a comment CID.
- Possibly **a community's cached settings** (title, rules, challenges), which measure about 5 KB. A full page of posts is too large to send as one item (see below), so accepted posts would have to be shared one at a time.

### Measured Sizes

Measured on 2026-11-14 with pkc-js 0.0.144. Publications were signed locally and never published. Sizes are bytes.

| Publication | JSON | CBOR | CBOR, key and signature as bytes | Fixed overhead |
|---|---|---|---|---|
| Post, 40-character title, no content | 450 | 408 | 375 | 335 |
| Post, 40-character title, 280 characters | 753 | 707 | 674 | 354 |
| Post, 40-character title, 1,000 characters | 1,473 | 1,427 | 1,394 | 354 |
| Reply, 20 characters | 576 | 523 | 490 | 470 |
| Reply, 280 characters | 836 | 785 | 752 | 472 |
| Vote | 482 | 435 | 402 | 402 |

Fixed overhead is the last column minus the text. Most of it is field names, the list of signed property names, and keys and CIDs written as text. A packed format that sends keys, CIDs and the signature as raw bytes and rebuilds the field names on arrival would need about 140 bytes for a post, 210 for a reply and 170 for a vote, plus the text. That is an estimate, not a measurement, and the receiver must be able to rebuild the exact signed fields or the signature will not verify.

bitchat's mesh sends a frame of up to 512 bytes whole and splits anything larger into fragments of 469 bytes. Its header takes 30 of those bytes. So, encoded as pkc-js does today:

- a vote fits in one frame as CBOR,
- a typical post or reply takes 2 fragments,
- a 1,000-character post takes 4.

Community data, from three live 5chan boards on the same day:

| Item | JSON | Compressed (deflate) |
|---|---|---|
| Community settings without posts | 4.6 to 5.0 KB | about 2 KB |
| Whole signed record with its preloaded page (246 to 634 comments) | 400 KB to 1.04 MB | 79 to 202 KB |
| One accepted comment with its community-signed update | about 1.5 KB median | not measured |

Text was short: the median comment had 24 to 38 characters. The whole record is signed as one piece, so it can only be verified whole. At 79 to 202 KB compressed that is 170 to 430 fragments, more than bitchat's 128 KiB limit for packet types it does not know and more than bitchat-android's 256 fragments per packet. Bluetooth transfer speed between phones is not measured yet.

### The Nearby View

- Shows publications that arrived over the mesh, after verifying their signatures with pkc-js, labelled as not accepted by their community yet.
- **It is unmoderated.** No community node has seen these posts, so no anti-spam challenges ran and no moderator acted. The client needs local blocking by author, rate limits and a way to hide the Nearby view entirely. Author keys cost nothing to make, so per-author limits alone don't stop spam.
- Authors show as addresses unless the phone already resolved their `.bso` names, since resolving them needs the internet.

### Back Online

- **The user's own queued posts:** the app signs each one again with a fresh timestamp and publishes it normally. The community's challenges apply at that point. Challenges the author can answer up front (`challengeRequest.challengeAnswers`, `challengeCommentCids`) can be supplied automatically; interactive ones such as image captchas need the user.
- Each queued post shows its state: queued, publishing, accepted or rejected.
- **Other people's nearby posts are never uploaded by this phone.** Only the author's own app can publish them.
- **Replies to nearby-only posts can't be published yet.** A reply names its parent's CID, and the community only produces a CID when it accepts a comment: it adds fields such as `depth` and `previousCid` before hashing ([`publication-store.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/runtime/node/community/local-community/publication-store.ts#L532-L539)). A reply has to wait until its parent's author publishes the parent and it is accepted. Votes on nearby-only posts have the same problem.

### Fit by Client

| Client | Fit |
|---|---|
| Profile-based clients, such as the Phase 3 Twitter/X alternative | Best. pkc-js models a profile as an [author community](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/docs/protocol/author-communities.md): a community addressed by the author's key that only its owner can post to. The owner's client signs, and the RPC never re-signs, so queued posts need nobody else's approval, and nearby readers can verify they came from that author. |
| 5chan, Seedit | Works, with caveats: nearby threads are unmoderated, and the board can challenge or reject queued posts on reconnect. |

### Privacy

- A phone with Nearby on advertises over Bluetooth continuously, so anyone scanning can see that a device running it is nearby.
- Every nearby publication carries the author's public key. Anyone in range can tie that Bitsocial identity to a place and time. bitchat's whitepaper names the same weakness for its stable peer IDs.
- Communities that show authors under pseudonymity aliases can't do that for nearby copies, which carry the author's own key.
- A separate account for Nearby limits the exposure. Whether the client should create one by default is an open question.

## Later: Longer Range

Later stages carry Bitsocial over LoRa, packet radio or other links reaching kilometres instead of metres. They are where the 5-minute rule bites: an internet gateway that relays a post hours after the author signed it will see it rejected.

### Mesh Transport

[Reticulum](https://reticulum.network/) is the working assumption. Each gateway, mesh-hosted community and client is a Reticulum destination. Announces make them reachable without addresses or DNS, and LXMF propagation nodes hold messages for peers who are offline. It runs over almost any carrier, down to a few bits per second with a 500-byte MTU, and defines no communities, posts or moderation. That is the part Bitsocial already has.

### Gateway

A normal Bitsocial node (pkc-js) with a mesh interface.

- **Downlink:** follows a configured list of communities and pushes compact snapshots into the mesh: community metadata plus recent posts, trimmed to a bandwidth budget.
- **Uplink:** relays a client's challenge request to the community's pubsub topic and relays the challenge and verification messages back, while a live path exists.
- **Trust:** the client builds and encrypts the challenge request itself (pkc-js encrypts it to the community's key), so the gateway carries an opaque, signed message. It cannot read, forge or alter the post. It can drop or delay it, which is why a mesh should be able to use more than one gateway.

### Mesh-Hosted Community

A community whose owner node sits on the mesh, for example a Raspberry Pi with a LoRa radio and a solar panel. Posts from mesh clients reach the owner node directly, with no internet hop, so the 5-minute window is only a matter of mesh latency. When the node has internet access it is also an ordinary Bitsocial community, reachable from 5chan, Seedit or any other client.

### Clocks

Off-grid devices drift. A device that has not synced time in a while can drift past 5 minutes on its own, so the gateway, or GPS on the radio, would have to provide time. Phones in Stage 1 don't have this problem, because they sign when they are back online.

### Carriers

| Carrier | Range | Notes |
|---|---|---|
| Bluetooth | Tens of metres per hop | Stage 1. Enough for text posts and small snapshots. Useful for a building, camp or protest. |
| Wi-Fi | Tens of metres | Fast enough to sync full snapshots. |
| LoRa (licence-free ISM bands) | Kilometres per hop | Throughput in bits to kilobits per second. Text only, small snapshots, pre-answered challenges. |
| Amateur packet radio (e.g. NPR on 70 cm) | Long, with repeaters | Needs an operator licence. Amateur rules generally prohibit encrypting traffic, which conflicts with Reticulum's default encryption and pkc-js's encrypted challenge messages. |
| Sneakernet (USB stick, SD card) | Anywhere | Works for reading snapshots. Publishing hits the 5-minute rule. |

## Not in Scope

- **Private chat.** bitchat and LXMF clients already do off-grid messaging, and Bitsocial has no native DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)).
- **A new radio layer.** Reuse bitchat's mesh design, Reticulum or another existing stack; do not build modems or SDR pipelines.
- **Media.** Images and video do not fit low-bandwidth links. Nearby posts and mesh snapshots would carry text and links only.
