# How It Could Work

A sketch, not a spec. It describes the pieces an off-grid Bitsocial would need and the one protocol constraint that shapes them.

## What Already Works Offline

Bitsocial data is self-verifying:

- Every publication (post, reply, vote, edit) is signed by its author's Ed25519 key.
- Every community is addressed by its owner's public key, and its state is signed by that key.

A mesh client can check a post or a community snapshot it received over LoRa exactly as it would one received over libp2p. Carrying Bitsocial data across a mesh needs no new trust model.

## The Constraint: Publications Expire After 5 Minutes

A community node rejects a publication whose timestamp is more than 5 minutes from its own clock, and drops challenge messages outside the same window. See `ERR_PUBLICATION_TIMESTAMP_IS_NOT_IN_PROPER_RANGE` in [`publication-validation.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/runtime/node/community/local-community/publication-validation.ts#L164) and the pubsub timestamp checks in [`signatures.ts`](https://github.com/pkcprotocol/pkc-js/blob/232e02a3e5d6a23a28aa08ea83e299cf21435ed2/src/signer/signatures.ts#L1027).

That splits the design into two cases:

| Case | Works with today's protocol? |
|---|---|
| **Live path.** The client can reach a gateway over the mesh right now, even through several hops, and the round trip takes well under 5 minutes. | Yes. The client signs at send time and the gateway relays immediately. |
| **Store-and-forward.** No gateway is reachable. The post waits on a propagation node for minutes, hours or days. | No. A post signed before it waits will be rejected when it arrives. Either the community runs on the mesh (below), or the protocol needs an opt-in rule for delayed publications ([open-questions.md](open-questions.md)). |

Off-grid clocks make the window tighter: a device that has not synced time in a while can drift past 5 minutes on its own. The gateway, or GPS on the radio, would have to provide time.

## Pieces

### Mesh Transport

Reticulum is the working assumption. Each gateway, mesh-hosted community and client is a Reticulum destination. Announces make them reachable without addresses or DNS, and LXMF propagation nodes hold messages for peers who are offline.

### Mesh Client

- **Reads** communities from what the mesh carries: snapshots pushed by a gateway, or a mesh-hosted community directly.
- **Writes** by signing a publication with the author's key at send time, when a live path exists. Without one, it keeps an unsigned draft and signs it once a path appears.
- **Answers challenges up front** where the community allows it. pkc-js already lets a publication carry `challengeRequest.challengeAnswers` and `challengeCommentCids`, which avoids a round trip on slow links. Interactive challenges such as image captchas cost a round trip and a lot of bytes.
- **Shows delivery state**: draft, sent to gateway, accepted or rejected by the community.
- Shows public keys instead of names unless a gateway resolved the name earlier, since `.bso` resolution needs the internet.

### Gateway

A normal Bitsocial node (pkc-js) with a mesh interface.

- **Downlink:** follows a configured list of communities and pushes compact snapshots into the mesh: community metadata plus recent posts, trimmed to a bandwidth budget.
- **Uplink:** relays a client's challenge request to the community's pubsub topic and relays the challenge and verification messages back over the mesh.
- **Trust:** the client builds and encrypts the challenge request itself (pkc-js encrypts it to the community's key), so the gateway carries an opaque, signed message. It cannot read, forge or alter the post. It can drop or delay it, which is why a mesh should be able to use more than one gateway.

### Mesh-Hosted Community (optional)

A community whose owner node sits on the mesh, for example a Raspberry Pi with a LoRa radio and a solar panel. Posts from mesh clients reach the owner node directly, with no internet hop, so the 5-minute window is only a matter of mesh latency. When the node has internet access it is also an ordinary Bitsocial community, reachable from 5chan, Seedit or any other client.

## Carriers

| Carrier | Range | Notes |
|---|---|---|
| Wi-Fi, Bluetooth | Tens of metres | Fast enough to sync full snapshots. Useful for a building, camp or protest. |
| LoRa (licence-free ISM bands) | Kilometres per hop | Throughput in bits to kilobits per second. Text only, small snapshots, pre-answered challenges. |
| Amateur packet radio (e.g. NPR on 70 cm) | Long, with repeaters | Needs an operator licence. Amateur rules generally prohibit encrypting traffic, which conflicts with Reticulum's default encryption and pkc-js's encrypted challenge messages. |
| Sneakernet (USB stick, SD card) | Anywhere | Works for reading snapshots. Publishing hits the 5-minute window. |

## Not in Scope

- **Private chat.** LXMF clients already do off-grid messaging, and Bitsocial has no native DMs yet ([pkc-js#51](https://github.com/pkcprotocol/pkc-js/issues/51)).
- **A new radio layer.** Reuse Reticulum or another existing stack; do not build modems or SDR pipelines.
- **Media.** Images and video do not fit low-bandwidth links. Mesh snapshots would carry text and links only.
