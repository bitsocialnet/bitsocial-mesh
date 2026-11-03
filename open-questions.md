# Open Questions

Design decisions not yet made. These are the questions to answer before Bitsocial Mesh moves from idea to spec.

## Stage 1: The Nearby Setting

### Publication Sizes

The first concrete task is measuring, as pkc-js encodes them:

- a signed post, reply and vote,
- a community's latest signed state, and one page of posts.

Those numbers decide how many Bluetooth fragments a post needs, whether cached community data can be shared nearby, and later whether LoRa is viable for posting, only for reading, or not at all.

### Wire Format

- Should Nearby use its own Bluetooth service and packet format, or bitchat's? Using bitchat's could let bitchat phones relay Bitsocial packets, but whether bitchat relays packet types it does not recognise is not checked.
- Reuse [bitchat-android](https://github.com/permissionlesstech/bitchat-android)'s Bluetooth code (GPL-3.0, Kotlin), port from the public-domain [iOS code](https://github.com/permissionlesstech/bitchat), or write from the [whitepaper](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md)? bitchat-android's licence file is GPL-3.0, but its README still says public domain and nothing says whether later GPL versions apply. If it is GPL-3.0 only, a client that includes it can't be distributed under a later GPL version.

### Retention

How long does a phone keep and re-share other people's nearby publications? bitchat keeps public chat for 6 hours and board notices for up to 7 days. Longer helps posts travel between groups; shorter limits spam and storage.

### Replies and Votes on Nearby-Only Posts

A reply or vote names a comment CID, and a comment only gets one when its community accepts it ([how-it-could-work.md](how-it-could-work.md#back-online)). Options:

- Hold nearby replies and votes until the parent appears in the community, then sign and publish them. Only works if the replying user's app is online after the parent's author published.
- Keep them nearby-only and never publish them.
- Disallow replying to nearby-only posts.

### Unmoderated Content

- Should the Nearby view be visible by default when the setting is on, or collapsed behind a warning?
- Author keys are free to create, so per-author rate limits don't stop spam. What else: per-connection limits, proof of work on each publication, or showing only authors the user already follows?
- Should 5chan and Seedit only show nearby posts for boards the user has joined?

### Privacy

- Should the client use a separate account for Nearby by default, so the main identity is never tied to a place?
- Can the Bluetooth identifiers rotate? bitchat lists rotating on-air identity as future work.

### Platforms and Testing

- Battery use and background reliability on Android vary by phone maker. They need measuring on real phones; emulator Bluetooth support is not checked.
- Can Electron take part on desktop, and with which Bluetooth library?
- iOS needs iOS builds of the clients first, and iOS limits Bluetooth advertising in the background.

## Later: Longer Range

### Delayed Publications

Community nodes reject publications more than 5 minutes old (see [how-it-could-work.md](how-it-could-work.md#the-5-minute-rule)). A gateway delivering a post the author signed hours earlier is impossible without changing that rule. Options:

- **Live paths only.** Only publish when a gateway is reachable in real time. No protocol change, but no posting from a fully disconnected mesh.
- **Mesh-hosted communities only.** Offline posting works for communities whose node is on the mesh. No protocol change, but internet communities stay read-only from the mesh.
- **A BSIP for delayed publications.** A community opts in to a longer window for relayed publications, in keeping with communities setting their own rules. Open sub-questions:
  - What stops backdating, if the window is hours or days? Is the author's timestamp enough, or does a gateway attest when it first saw the publication?
  - How are delayed replies shown in threads that moved on without them?
  - Do challenge messages need the same exception, or does the gateway run a fresh exchange with a pre-answered request?

### Snapshot Format

- Does the gateway push raw IPFS blocks, verifiable by CID, or a compact re-encoding that keeps the original signed fields so signatures still verify?
- Can snapshots be sent as diffs against what the mesh already has?
- Who picks which communities a gateway carries, and how many posts fit in a budget?

### Challenges Over Slow Links

Every community chooses its own anti-spam challenges. Over a mesh, they fall into three groups:

- checks that need nothing from the author,
- challenges the author can answer up front with `challengeAnswers` or `challengeCommentCids`,
- interactive challenges such as image captchas, which cost a round trip and many bytes.

Should clients warn when a community's challenges can't be completed over the current link? Would communities want a mesh-friendly challenge setup?

### Transport

- **Reticulum** is the working assumption. Its [reference implementation](https://github.com/markqvist/Reticulum) is Python and pkc-js is TypeScript: run Reticulum as a separate local daemon, or something else?
- **Licence.** The [Reticulum License](https://reticulum.network/manual/license.html) adds use restrictions to MIT-style terms and is not an OSI licence. Is running it as a separate program enough to keep this GPL-3.0 project clean, or does that need legal review? Reticulum's maintainers also warn that several unofficial, machine-generated ports carry license claims they consider void; use the reference implementation or programs listed in its manual.
- **Alternatives to evaluate:** Meshtastic for LoRa.

### Radio Rules

- Amateur bands generally prohibit encrypted traffic. In the US, 47 CFR 97.113(a)(4) prohibits messages encoded to obscure their meaning. Bitsocial publications are public and signed, which is allowed, but pkc-js challenge messages and Reticulum links are encrypted. Is an amateur-band mode worth designing, or should the project target licence-free bands only?
- Licence-free bands have their own limits, such as duty cycles in parts of the EU 868 MHz band.

### Gateways and Spam

- Gateways relay for anyone on the mesh, and mesh spam costs everyone bandwidth. Should gateways require LXMF proof-of-work stamps or rate limits before relaying? Community challenges still apply at the community node, but only after the bytes crossed the mesh.
- How does a client choose between gateways, and detect one that drops its posts?

### Clocks

Off-grid devices drift. Where does a mesh client get time accurate enough for the 5-minute window: the gateway, GPS on the radio, or Reticulum itself?

### First Hardware Target

The likely first long-range target is a laptop or phone paired with a LoRa radio, plus a Raspberry Pi gateway. This waits until the publication sizes are measured.
