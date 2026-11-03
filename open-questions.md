# Open Questions

Design decisions not yet made. These are the questions to answer before Bitsocial Mesh moves from idea to spec.

## Delayed Publications — The Biggest Open Question

Community nodes reject publications more than 5 minutes old (see [how-it-could-work.md](how-it-could-work.md#the-constraint-publications-expire-after-5-minutes)). True store-and-forward posting to an internet community is impossible without changing that rule. Options:

- **Live paths only.** Only publish when a gateway is reachable in real time. No protocol change, but no posting from a fully disconnected mesh.
- **Mesh-hosted communities only.** Offline posting works for communities whose node is on the mesh. No protocol change, but internet communities stay read-only from the mesh.
- **A BSIP for delayed publications.** A community opts in to a longer window for relayed publications, in keeping with communities setting their own rules. Open sub-questions:
  - What stops backdating, if the window is hours or days? Is the author's timestamp enough, or does a gateway attest when it first saw the publication?
  - How are delayed replies shown in threads that moved on without them?
  - Do challenge messages need the same exception, or does the gateway run a fresh exchange with a pre-answered request?

## Bandwidth

The first concrete task is measuring, as pkc-js encodes them:

- a signed comment, reply and vote,
- an encrypted challenge request, with and without pre-supplied answers,
- a community snapshot: metadata plus the latest N posts.

Those numbers decide whether LoRa is viable for posting, only for reading, or not at all.

## Snapshot Format

- Does the gateway push raw IPFS blocks, verifiable by CID, or a compact re-encoding that keeps the original signed fields so signatures still verify?
- Can snapshots be sent as diffs against what the mesh already has?
- Who picks which communities a gateway carries, and how many posts fit in a budget?

## Challenges Over Slow Links

Every community chooses its own anti-spam challenges. Over a mesh, they fall into three groups:

- checks that need nothing from the author,
- challenges the author can answer up front with `challengeAnswers` or `challengeCommentCids`,
- interactive challenges such as image captchas, which cost a round trip and many bytes.

Should clients warn when a community's challenges can't be completed over the current link? Would communities want a mesh-friendly challenge setup?

## Transport

- **Reticulum** is the working assumption. Its [reference implementation](https://github.com/markqvist/Reticulum) is Python and pkc-js is TypeScript: run Reticulum as a separate local daemon, or something else?
- **Licence.** The [Reticulum License](https://reticulum.network/manual/license.html) adds use restrictions to MIT-style terms and is not an OSI licence. Is running it as a separate program enough to keep this GPL-3.0 project clean, or does that need legal review? Reticulum's maintainers also warn that several unofficial, machine-generated ports carry license claims they consider void; use the reference implementation or programs listed in its manual.
- **Alternatives to evaluate:** Meshtastic for LoRa, and Bluetooth mesh designs such as bitchat, which bridges to Nostr for internet reach.

## Radio Rules

- Amateur bands generally prohibit encrypted traffic. In the US, 47 CFR 97.113(a)(4) prohibits messages encoded to obscure their meaning. Bitsocial publications are public and signed, which is allowed, but pkc-js challenge messages and Reticulum links are encrypted. Is an amateur-band mode worth designing, or should the project target licence-free bands only?
- Licence-free bands have their own limits, such as duty cycles in parts of the EU 868 MHz band.

## Gateways and Spam

- Gateways relay for anyone on the mesh, and mesh spam costs everyone bandwidth. Should gateways require LXMF proof-of-work stamps or rate limits before relaying? Community challenges still apply at the community node, but only after the bytes crossed the mesh.
- How does a client choose between gateways, and detect one that drops its posts?

## Clocks

Off-grid devices drift. Where does a mesh client get time accurate enough for the 5-minute window: the gateway, GPS on the radio, or Reticulum itself?

## First Hardware Target

The likely first target is a laptop or phone paired with a LoRa radio, plus a Raspberry Pi gateway. This waits until the bandwidth numbers exist.
