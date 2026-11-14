# Open Questions

Design decisions not yet made. These are the questions to answer before Bitsocial Mesh moves from idea to spec.

## The Mesh

### Encoding

Sizes are now measured (see [how-it-could-work.md](how-it-could-work.md#measured-sizes)). A typical post or reply is 550 to 850 bytes as pkc-js encodes it, of which 350 to 470 bytes are fixed overhead.

- Send publications as pkc-js encodes them (2 fragments for a typical post), or in a packed format that could fit a short post in one frame? Packing saves airtime but adds a second encoding that must rebuild the signed fields exactly.
- A community's whole signed record is 79 to 202 KB compressed and only verifies whole. Should Nearby share accepted posts one at a time instead, each with its community-signed update (about 1.5 KB)?

### Wire Format

Checked on 2026-11-14 by reading the source of [bitchat](https://github.com/permissionlesstech/bitchat) at `04995f68` and [bitchat-android](https://github.com/permissionlesstech/bitchat-android) at `14ab73c4`. Nothing was run on phones.

- **Both apps relay packet types they do not recognise.** The type byte is not checked when a packet is decoded, and a packet that is not addressed to the phone goes to the relay step whatever its type. Unknown types need no signature and no prior announce from the sender.
- Limits that apply anyway: the version byte must be 1 or 2, the payload at most 128 KiB, duplicates are dropped, and the iOS app drops packets whose timestamp is more than 120 seconds from its clock.
- Neither project promises this. A later release could filter unknown types, or assign the type byte Nearby picked.

Still to decide:

- Use bitchat's Bluetooth service, so bitchat phones relay Bitsocial packets, or an own service? Sharing theirs adds foreign traffic to their mesh, so it should be agreed with bitchat's maintainers first. **Proposed:** adopt bitchat's packet format and fragmentation either way, keep the service identifier a build setting, start on an own service, and switch to bitchat's only with their agreement.
- Reuse bitchat-android's Bluetooth code, port the public-domain iOS code, or write from the [whitepaper](https://github.com/permissionlesstech/bitchat/blob/main/WHITEPAPER.md)? **Proposed:** reuse bitchat-android's, since Android is the first target.
- **Licence.** bitchat-android's `LICENSE.md` is the GPL version 3 text, changed from MIT on 2026-02-28. No file says "only" or "or later", and its README and privacy policy still say public domain. 5chan, Seedit and bitsocial-react-hooks are GPL-3.0-or-later, so a build that includes this code would be distributed under GPL version 3. That reading is not legal advice. Asking bitchat-android's maintainers to fix the README would settle it.

### Retention

How long does a phone keep and re-share other people's nearby publications? bitchat keeps public chat for 6 hours and board notices for up to 7 days. Longer helps posts travel between groups; shorter limits spam and storage.

### Other Carriers

- Does the existing stack already work between devices on a Wi-Fi network with no internet behind it? If it does, a laptop hotspot is a faster carrier than Bluetooth and needs far less new code.
- Wi-Fi Aware and Wi-Fi Direct connect Android phones without an access point. Worth it, or Bluetooth only?

### Platforms and Testing

- Battery use and background reliability on Android vary by phone maker. They need measuring on real phones; emulator Bluetooth support is not checked.
- Can Electron take part on desktop, and with which Bluetooth library?
- iOS needs iOS builds of the clients first, and iOS limits Bluetooth advertising in the background.

## A Community on the Mesh

### What Breaks With No Internet

Not tested. A host and its clients may depend on the internet for name resolution, for the HTTP routers that find peers, for bootstrap peers and for challenges that call outside services. Which of these stop a community from working on an isolated network, and which only log errors?

### The Host

- Hosting needs Node.js today, so a phone can't host. Is a phone host worth pursuing, or is "someone nearby has a laptop" good enough?
- The host is one device and one person. If it is switched off, seized or out of battery, the community stops. Can a second device take over, and how would it hold the community's key safely?
- The person running the host is easy to locate by radio. What can reduce that risk?

### The Bridge

- The publishing exchange is 4 messages. How many Bluetooth packets and how many seconds does it take across several hops? It has to finish well inside 5 minutes.
- How does a phone get the posts it missed: ask the host for everything since a time, or sync between neighbouring phones as bitchat does?
- Phones can verify each accepted post and the host's signed update for it. Can they also tell that the host is withholding posts?

### Finding and Trusting a Community

- How does a phone learn which communities are hosted nearby?
- Anyone can announce a community with the title of a well-known one. Addresses are keys, so a phone that saw the real community before can tell them apart. What does a first-time user see?
- A community created during a blackout has no history and no reputation. Should clients label it as local and new?

### Challenges

Challenges that need an outside service, such as a captcha image fetched from the web, can't run. Which of the existing challenges work with no internet, and does a host need a fallback set for a blackout?

## Delayed Delivery

Community hosts reject publications more than 5 minutes old (see [how-it-could-work.md](how-it-could-work.md#the-5-minute-rule)). Accepting a post that was signed hours earlier and carried out by someone else needs a change to that rule, proposed as a BSIP and decided by the pkc-js maintainers. A community would opt in to a longer window for delayed publications, in keeping with communities setting their own rules.

- What stops backdating, if the window is hours or days? Is the author's timestamp enough, or does the carrier attest when it first saw the publication?
- Does the carrier deliver the author's sealed request as it is, which means the challenge messages need the same exception, or does it run a fresh exchange on the author's behalf?
- The result of the exchange goes to the carrier. How does the author learn whether the post was accepted?
- Several carriers may deliver the same post. The host has to accept it once.
- How are delayed replies shown in threads that moved on without them?
- A carrier holds other people's sealed posts. What does it reveal about the carrier if their phone is searched?

## The Nearby Feed

### Replies and Votes on Nearby-Only Posts

A reply or vote names a comment CID, and a comment only gets one when its community accepts it ([how-it-could-work.md](how-it-could-work.md#back-online)). Options:

- Hold nearby replies and votes until the parent appears in the community, then sign and publish them. Only works if the replying user's app is online after the parent's author published.
- Keep them nearby-only and never publish them.
- Disallow replying to nearby-only posts.

### Unmoderated Content

- Should the Nearby view be visible by default when the setting is on, or collapsed behind a warning?
- Author keys are free to create, so per-author rate limits don't stop spam. What else: per-connection limits, proof of work on each publication, or showing only authors the user already follows?
- Should 5chan and Seedit only show nearby posts for boards the user has joined?

## Privacy

- Should the client use a separate account for Nearby by default, so the main identity is never tied to a place?
- Can the Bluetooth identifiers rotate? bitchat lists rotating on-air identity as future work.
- A blackout is when being located is most dangerous. Should the mesh be off unless the user turns it on each time, and should the app say plainly what nearby devices can see?

## Getting the App to People

- Nobody can download the app once the internet is gone. Can the Android app send its own install file to a nearby phone?
- App stores can remove an app in a country at a government's request. Is a direct download the main channel?

## A Map

A world map of communities by coarse location is possible while online, the way bitchat's maps read location channels from Nostr relays. Open points: what a community publishes to appear on it, how coarse the location is, and whether the feature is safe at all for the people this project is for.

## Later: Longer Range

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

The likely first long-range target is a laptop or phone paired with a LoRa radio, plus a Raspberry Pi gateway. The [measured sizes](how-it-could-work.md#measured-sizes) say a post fits in one or two 500-byte Reticulum packets, while a board's whole record does not suit a link measured in bits to kilobits per second. Posting looks viable; reading needs a much smaller snapshot format.
