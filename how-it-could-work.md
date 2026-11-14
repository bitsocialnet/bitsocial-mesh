# How It Could Work

A sketch, not a spec. The goal is that Bitsocial communities keep working with no internet. This describes what already holds offline, what does not, and the four stages that close the gap, in the order they would be built.

## What Already Works Offline

Bitsocial data is self-verifying:

- Every publication (post, reply, vote, edit) is signed by its author's Ed25519 key.
- Every community is addressed by its owner's public key, and its state is signed by that key.

A phone can check a post or a community snapshot it received over Bluetooth or LoRa exactly as it would one received over libp2p. Carrying Bitsocial data across a mesh needs no new trust model.

## What Does Not: The Community's Host

A signed post is not yet part of a community. The community's host node has to accept it, and only the host can:

- run the community's anti-spam challenge,
- give the post its CID, which replies and votes need in order to name it. The host adds fields such as `depth` and `previousCid` before hashing,
- apply moderation,
- sign the community's updated state.

So with the host out of reach, people can exchange signed posts but nothing is accepted, threaded or moderated. This is the gap a blackout exposes, and it is why Bitsocial can't simply pass posts from phone to phone the way a mesh chat does.

### The 5-Minute Rule

Every publication carries the time its author signed it. When it arrives, the host compares that time with its own clock and rejects the publication if the two differ by more than 5 minutes ([`publication-validation.ts`](https://github.com/pkcprotocol/pkc-js/blob/25a3066e0/src/runtime/node/community/local-community/publication-validation.ts#L164)). The messages of the publishing exchange are checked against the same window by whoever receives them ([`signatures.ts`](https://github.com/pkcprotocol/pkc-js/blob/25a3066e0/src/signer/signatures.ts#L1036-L1044)).

The effect is that a post has to be delivered when it is written. The author can sign it again later, because they hold the key. Nobody else can, so a post written today can't be handed to the community tomorrow by another person.

| Who publishes, and when | Accepted with today's protocol? |
|---|---|
| The author's app, to a host it can reach directly | Yes. This is [Stage 1](#stage-1-the-community-box). |
| The author's own app, once it is back online, signing at that moment | Yes. It is an ordinary publication. |
| Anyone delivering a post the author signed earlier | No, once it is more than 5 minutes old. This is [Stage 2](#stage-2-delayed-delivery). |

## Stage 1: The Community Box

A laptop or a small computer hosts the community and opens a Wi-Fi hotspot. People nearby join the hotspot and use the community as they always do. Posts reach the host directly, so nothing about the protocol changes.

### The Host

Hosting a community needs Node.js and a data directory: pkc-js's `LocalCommunity` is [Node-only](https://github.com/pkcprotocol/pkc-js/blob/25a3066e0/docs/protocol/community-architecture.md). So the box is a laptop running the desktop app, or a small computer such as a Raspberry Pi. A phone can't be the host today.

### Reaching It

- **Over the hotspot.** Wi-Fi is fast enough for everything a client normally loads, so there is no special sync format and nothing to fragment.
- **With nothing but a browser.** If the box also serves the web client, someone who never installed anything joins the hotspot and opens a page. This would also cover iPhones, which have no app. It is untested: browsers restrict what a page loaded from a local address without HTTPS may do, and how the page should talk to the box's node safely is not designed.
- **With the installed app.** An app that is already on the phone only needs to find the box on the local network.

### Which Communities Keep Working

| Community | In a blackout |
|---|---|
| One whose host is the box | Keeps working, for everyone on the hotspot. |
| A new one, created on the box during the blackout | Works locally from the start, and is an ordinary Bitsocial community once it has a link out. |
| One whose host is somewhere else | Unreachable. The box can show what it cached. New posts wait for [Stage 2](#stage-2-delayed-delivery) or for the internet. |

### When a Link Returns

The box is an ordinary Bitsocial node. Everything it accepted during the blackout is already part of the community, so it publishes its state and the rest of the network sees it. Nothing is signed again and no CID changes.

### Not Yet Known

Which parts of a host stop working with no internet has not been tested. Candidates are name resolution (`.bso` names can't be looked up offline), the HTTP routers used to find peers, and any challenge that calls an outside service. Finding out is the first task on the [roadmap](ROADMAP.md).

## Stage 2: Delayed Delivery

For communities hosted outside the affected area, the only way in is for someone to carry posts out: a phone that crosses a border, reaches a working network or gets a few minutes of signal. Today the host would reject those posts as too old.

How it could work:

1. The author writes and signs a post with no internet. Their app hands the signed post to other devices nearby, over the box's hotspot or any other link.
2. A carrier's app holds it. The post is public and signed, so the carrier can read it but can't change it.
3. When the carrier reaches the internet, their app runs the ordinary publishing exchange with the community, live, with the author's post inside.
4. The community accepts it, because it has opted in to a longer window for the post's own timestamp.

Only one check has to change: the host's comparison of the post's timestamp with its clock. The exchange around it is fresh, so its own 5-minute checks still hold. Communities that don't opt in behave exactly as today, and clients that know nothing about this keep working. It is still a protocol change, so it has to be proposed and decided by the pkc-js maintainers.

What makes it hard:

- **Backdating.** A long window lets an author claim a post was written earlier than it was.
- **Challenges.** There is no author on the line to solve a captcha. Only challenges the author can answer up front (`challengeRequest.challengeAnswers`, `challengeCommentCids`) work, and the carrier has to be given those answers.
- **The result goes to the carrier,** not the author.
- **Replies arrive late,** in threads that moved on without them, and a reply can only name a parent that already has a CID.

These are in [open-questions.md](open-questions.md#delayed-delivery).

## Stage 3: Linking Boxes

A hotspot covers a building or a square. To reach across a city, boxes need links to each other, and LoRa radios reach kilometres per hop on licence-free bands. The radios are run by whoever runs the boxes; people joining a box still only need Wi-Fi.

With links between boxes:

- someone on one box's hotspot can use a community hosted on another box,
- a box that still has internet can relay for the ones that don't.

LoRa is slow, so this stage carries posts and small updates, not whole boards.

### Mesh Transport

[Reticulum](https://reticulum.network/) is the working assumption. Each box, gateway and client is a Reticulum destination. Announces make them reachable without addresses or DNS, and LXMF propagation nodes hold messages for peers who are offline. It runs over almost any carrier, down to a few bits per second with a 500-byte MTU, and defines no communities, posts or moderation. That is the part Bitsocial already has.

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
| Bluetooth | Tens of metres per hop | Stage 4. Enough for text posts. |
| Wi-Fi hotspot | Tens of metres | Stage 1. Fast enough for everything, including a board's whole record. |
| LoRa (licence-free ISM bands) | Kilometres per hop | Throughput in bits to kilobits per second. Text only, small snapshots, pre-answered challenges. |
| Amateur packet radio (e.g. NPR on 70 cm) | Long, with repeaters | Needs an operator licence. Amateur rules generally prohibit encrypting traffic, which conflicts with Reticulum's default encryption and pkc-js's encrypted challenge messages. |
| Sneakernet (USB stick, SD card) | Anywhere | Works for reading snapshots. Publishing hits the 5-minute rule. |

## Stage 4: Phone to Phone

Phones exchange posts directly over Bluetooth with no box in range. It comes last because it is the most new code for the shortest range, and because what it delivers is weaker: with no host reachable, nothing a phone receives has been accepted by any community. It may not be built.

### The Mesh

Every phone is both a Bluetooth central and peripheral and relays packets for others, as in [bitchat](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md):

- a hop limit on every packet, so floods die out,
- duplicate suppression by a hash of each packet,
- a short random delay before relaying, so duplicates are dropped before they are sent,
- fragmentation for publications larger than a Bluetooth packet,
- periodic sync between neighbours of what each phone holds, so a phone that walks from one group to another carries posts with it.

On Android the app runs a foreground service so it keeps advertising and scanning with the screen off. Battery cost and how aggressively phone makers kill background services are unmeasured; both need testing on real devices.

### The Nearby Feed

What phones would exchange:

- **Publications the user writes while it is on:** posts and replies, signed by the author's key as usual.
- **Votes**, which only matter on comments a community has already accepted, since a vote names a comment CID.

What the user sees:

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

## Measured Sizes

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

What the sizes mean for each carrier: over Wi-Fi none of this matters. Over LoRa a post fits in one or two packets, but a board's whole record does not suit a link measured in bits to kilobits per second. Over Bluetooth posts are fine and whole records are not.

## Privacy

- **On a box's hotspot,** the box sees every device that joins and everything it posts. Whoever runs the box is trusted the way a community's owner already is, plus they know who was physically there.
- **The box is a radio beacon.** A hotspot can be found by anyone looking for it, and so can the person running it.
- Every publication carries the author's public key. Anyone who receives it can tie that Bitsocial identity to a place and time.
- A separate account for use on a box limits the exposure. Whether the client should create one by default is an open question.
- A phone on a Bluetooth mesh (Stage 4) advertises continuously, so anyone scanning can see that a device running it is nearby. bitchat's whitepaper names the same weakness for its stable peer IDs.
- In a shutdown these matter more than usual. The people most likely to use this are the people most at risk from being located.

## Not in Scope

- **Private chat.** bitchat and LXMF clients already do off-grid messaging, and Bitsocial has no native DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)).
- **A new radio layer.** Reuse Reticulum, bitchat's mesh design or another existing stack; do not build modems or SDR pipelines.
- **Media over slow links.** Images and video do not fit LoRa or Bluetooth. On a box's hotspot, media a box already holds could be served, but links to the outside web will not load.
