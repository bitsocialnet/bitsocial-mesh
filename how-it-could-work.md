# How It Could Work

A sketch, not a spec. The goal is that Bitsocial communities keep working with no internet. This describes what already holds offline, what does not, the two capabilities that close the gap, and the smaller Nearby feed that needs no protocol change.

## What Already Works Offline

Bitsocial data is self-verifying:

- Every publication (post, reply, vote, edit) is signed by its author's Ed25519 key.
- Every community is addressed by its owner's public key, and its state is signed by that key.

A phone can check a post or a community snapshot it received over Bluetooth or LoRa exactly as it would one received over libp2p. Carrying Bitsocial data across a mesh needs no new trust model.

## What Does Not: The Community's Host

A signed post is not yet part of a community. The community's host node has to accept it, and only the host can:

- run the community's anti-spam challenge,
- give the post its CID, which replies and votes need in order to name it. The host adds fields such as `depth` and `previousCid` before hashing ([`publication-store.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/runtime/node/community/local-community/publication-store.ts#L532-L539)),
- apply moderation,
- sign the community's updated state.

So with the host out of reach, people can exchange signed posts but nothing is accepted, threaded or moderated. This is the gap a blackout exposes.

### The 5-Minute Rule

The host also rejects a publication whose timestamp is more than 5 minutes from its own clock, and drops challenge messages outside the same window. See `ERR_PUBLICATION_TIMESTAMP_IS_NOT_IN_PROPER_RANGE` in [`publication-validation.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/runtime/node/community/local-community/publication-validation.ts#L164) and the pubsub timestamp checks in [`signatures.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/signer/signatures.ts#L1027).

| Who publishes, and when | Accepted with today's protocol? |
|---|---|
| The author's app, over a mesh path to a host that is on the mesh | Yes, if the round trip is well under 5 minutes. |
| The author's own app, once it is back online, signing at that moment | Yes. It is an ordinary publication. |
| Anyone delivering a post the author signed earlier | No, once it is more than 5 minutes old. |

The first row is [a community on the mesh](#capability-1-a-community-on-the-mesh). The second is the [Nearby feed](#the-nearby-feed). The third is [delayed delivery](#capability-2-delayed-delivery), and it needs a protocol change.

## The Mesh

Every phone is both a Bluetooth central and peripheral and relays packets for others, as in [bitchat](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md):

- a hop limit on every packet, so floods die out,
- duplicate suppression by a hash of each packet,
- a short random delay before relaying, so duplicates are dropped before they are sent,
- fragmentation for publications larger than a Bluetooth packet,
- periodic sync between neighbours of what each phone holds, so a phone that walks from one group to another carries posts with it.

On Android the app runs a foreground service so it keeps advertising and scanning with the screen off. Battery cost and how aggressively phone makers kill background services are unmeasured; both need testing on real devices.

Bluetooth is the first carrier, not the only one. A Wi-Fi network with no internet behind it, such as a laptop's hotspot, is far faster, and the existing libp2p stack may already work across one. That is not checked.

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

## Capability 1: A Community on the Mesh

The host node sits inside the affected area and devices reach it over the mesh. Posts get to the host directly, so the 5-minute window is only a matter of mesh latency, and the community works as it always does.

### The Host

Hosting a community needs Node.js and a data directory: pkc-js's `LocalCommunity` is [Node-only](https://github.com/pkcprotocol/pkc-js/blob/837ca7dbb/docs/protocol/community-architecture.md). So the host is a desktop app, a laptop or a small computer such as a Raspberry Pi, not a phone. Phones are clients.

### The Bridge

A piece beside the host that speaks to the mesh:

- **Posting.** Publishing is a 4-message encrypted exchange between the author's app and the host: request, challenge, answer, verification. The bridge carries those messages between mesh packets and the host. The app encrypts its request to the community's key, so phones that relay it carry an opaque message they can't read or alter.
- **Reading.** A community's whole signed record is too large for Bluetooth (see [Measured Sizes](#measured-sizes)). The bridge would send accepted posts one at a time, each with the host's signed update, about 1.5 KB per post.
- **Finding it.** The host announces itself on the mesh. A community's address is a key, so no DNS is needed, but `.bso` names can't be resolved without the internet and would show as keys unless the phone resolved them earlier.

### Which Communities Keep Working

| Community | In a blackout |
|---|---|
| One whose host is inside the affected area | Keeps working, for everyone who can reach it over the mesh. |
| A new one, created during the blackout on someone's laptop | Works locally from the start, and is an ordinary Bitsocial community once it has a link out. |
| One whose host is outside | Unreachable. Phones can show what they cached. New posts wait for [delayed delivery](#capability-2-delayed-delivery) or for the internet. |

### When a Link Returns

The host is an ordinary Bitsocial node. Everything it accepted during the blackout is already part of the community, so it publishes its state and the rest of the network sees it. Nothing is signed again and no CID changes.

### Not Yet Known

Which parts of a host stop working with no internet has not been tested. Candidates are name resolution, the HTTP routers used to find peers, and any challenge that calls an outside service. Finding out is the first task on the [roadmap](ROADMAP.md).

## Capability 2: Delayed Delivery

For communities whose host is outside the affected area, the only way in is for someone to carry posts out: a phone that crosses a border, reaches a working Wi-Fi network or gets a few minutes of signal. Today the host would reject those posts as too old.

What it would take:

- **A longer window, chosen by the community.** A community opts in to accepting publications signed hours or days earlier. This fits how Bitsocial works, since each community sets its own rules, but it is a protocol change and has to be proposed as a BSIP.
- **Something a carrier can deliver.** The author's app builds the encrypted request while offline. Any device that reaches the internet hands it to the community. The carrier can't read or change it.
- **Challenges answered in advance.** There is no author on the line to solve a captcha. Only challenges the author can answer up front (`challengeRequest.challengeAnswers`, `challengeCommentCids`) work, so a community that wants delayed posts needs a challenge setup that allows them.

What makes it hard: a long window lets an author backdate a post, the result of the exchange goes to the carrier instead of the author, and replies arrive in threads that moved on without them. These are in [open-questions.md](open-questions.md#delayed-delivery).

## The Nearby Feed

The piece that needs no protocol change and no host. It is a first step, not the goal: while the internet is down, nothing in it has been accepted by any community.

### What Goes Over the Air

- **Publications the user writes while it is on:** posts and replies, signed by the author's key as usual.
- **Votes**, which only matter on comments a community has already accepted, since a vote names a comment CID.

### The Nearby View

- Shows publications that arrived over the mesh, after verifying their signatures with pkc-js, labelled as not accepted by their community yet.
- **It is unmoderated.** No host has seen these posts, so no challenges ran and no moderator acted. The client needs local blocking by author, rate limits and a way to hide the view entirely. Author keys cost nothing to make, so per-author limits alone don't stop spam.

### Back Online

- **The user's own queued posts:** the app signs each one again with a fresh timestamp and publishes it normally. The community's challenges apply at that point. The copy nearby phones saw and the copy the community accepts are therefore different publications.
- Each queued post shows its state: queued, publishing, accepted or rejected.
- **Other people's nearby posts are never uploaded by this phone.** Only the author's own app can publish them.
- **Replies to nearby-only posts can't be published yet,** because the parent has no CID until its community accepts it. Votes on nearby-only posts have the same problem.

### Fit by Client

| Client | Fit |
|---|---|
| Profile-based clients, such as the [Phase 3 Twitter/X alternative](https://bitsocial.net/#master-plan-phase-3) | Best. pkc-js models a profile as an [author community](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/docs/protocol/author-communities.md): a community addressed by the author's key that only its owner can post to. Queued posts need nobody else's approval, and nearby readers can verify they came from that author. |
| 5chan, Seedit | Works, with caveats: nearby threads are unmoderated, and the board can challenge or reject queued posts on reconnect. |

## Privacy

- A phone on the mesh advertises over Bluetooth continuously, so anyone scanning can see that a device running it is nearby.
- Every publication carries the author's public key. Anyone in range can tie that Bitsocial identity to a place and time. bitchat's whitepaper names the same weakness for its stable peer IDs.
- Communities that show authors under pseudonymity aliases can't do that for copies sent directly between phones, which carry the author's own key.
- A separate account for the mesh limits the exposure. Whether the client should create one by default is an open question.
- In a shutdown these matter more than usual. The people most likely to use this are the people most at risk from being located.

## Later: Longer Range

Later stages carry Bitsocial over LoRa, packet radio or other links reaching kilometres instead of metres.

### Mesh Transport

[Reticulum](https://reticulum.network/) is the working assumption. Each gateway, mesh-hosted community and client is a Reticulum destination. Announces make them reachable without addresses or DNS, and LXMF propagation nodes hold messages for peers who are offline. It runs over almost any carrier, down to a few bits per second with a 500-byte MTU, and defines no communities, posts or moderation. That is the part Bitsocial already has.

### Gateway

A normal Bitsocial node (pkc-js) with a mesh interface.

- **Downlink:** follows a configured list of communities and pushes compact snapshots into the mesh: community metadata plus recent posts, trimmed to a bandwidth budget.
- **Uplink:** relays a client's challenge request to the community's pubsub topic and relays the challenge and verification messages back, while a live path exists.
- **Trust:** the client builds and encrypts the challenge request itself (pkc-js encrypts it to the community's key), so the gateway carries an opaque, signed message. It cannot read, forge or alter the post. It can drop or delay it, which is why a mesh should be able to use more than one gateway.

### Clocks

Off-grid devices drift. A device that has not synced time in a while can drift past 5 minutes on its own, so the gateway, or GPS on the radio, would have to provide time. The same applies to a host on the mesh with no battery-backed clock, such as a Raspberry Pi.

### Carriers

| Carrier | Range | Notes |
|---|---|---|
| Bluetooth | Tens of metres per hop | The first carrier. Enough for text posts and small snapshots. Useful for a building, camp or protest. |
| Wi-Fi | Tens of metres | Fast enough to sync full snapshots. |
| LoRa (licence-free ISM bands) | Kilometres per hop | Throughput in bits to kilobits per second. Text only, small snapshots, pre-answered challenges. |
| Amateur packet radio (e.g. NPR on 70 cm) | Long, with repeaters | Needs an operator licence. Amateur rules generally prohibit encrypting traffic, which conflicts with Reticulum's default encryption and pkc-js's encrypted challenge messages. |
| Sneakernet (USB stick, SD card) | Anywhere | Works for reading snapshots. Publishing hits the 5-minute rule. |

## Not in Scope

- **Private chat.** bitchat and LXMF clients already do off-grid messaging, and Bitsocial has no native DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)).
- **A new radio layer.** Reuse bitchat's mesh design, Reticulum or another existing stack; do not build modems or SDR pipelines.
- **Media.** Images and video do not fit low-bandwidth links. Posts on the mesh would carry text and links only.
