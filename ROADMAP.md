# Roadmap

**Status: not started.** Nothing below is scheduled. P2P chat on Bitsocial comes first, and this work only begins when someone picks up Phase 0.

The durations are rough guesses for one developer working on it full time, made before anything was measured. Phases 0 and 1 exist to replace them with real numbers. Together they come to about 2 to 3 months for an experimental Android release in one client.

Each phase ends with a check. If the check fails, the work stops there and the reason is written into [open-questions.md](open-questions.md).

## Phase 0: Measure and Decide (1 to 2 days)

Needs no phones.

- [ ] Measure the encoded size of a signed post, reply and vote as pkc-js produces them.
- [ ] Measure a community's latest signed state and one page of posts.
- [ ] Check whether bitchat phones relay packet types they do not recognise.
- [ ] Confirm bitchat-android's licence terms (GPL-3.0 only, or later versions too).
- [ ] Decide the wire format: an own Bluetooth service and packet format, or bitchat's.
- [ ] Decide whether to reuse bitchat-android's Bluetooth code, port the iOS code, or write from the whitepaper.

**Check:** the sizes are in [how-it-could-work.md](how-it-could-work.md), and the number of Bluetooth fragments a typical post needs is known.

## Phase 1: Two Phones (1 to 2 weeks)

Needs two physical Android phones, ideally from different makers. Emulators are not known to support this.

- [ ] Capacitor plugin skeleton: advertise, scan, and act as both central and peripheral.
- [ ] Send one signed post from phone A to phone B and verify it with pkc-js in the app.
- [ ] Keep it running with the screen off through a foreground service.
- [ ] Measure battery use over several hours on both phones.

**Check (go or no-go):** posts cross reliably with both screens off, and the battery cost is one a user would accept. This is the largest technical risk in the project.

## Phase 2: The Mesh (3 to 4 weeks)

Needs three or more phones.

- [ ] Fragmentation and reassembly for publications larger than one packet.
- [ ] Hop limit, duplicate suppression and a random delay before relaying.
- [ ] Periodic sync between neighbours of what each phone holds.
- [ ] Retention: decide and implement how long other people's publications are kept and re-shared.
- [ ] Per-connection rate limits.

**Check:** a post written on phone A reaches phone C through phone B, and a phone carried from one group to another delivers what it holds.

## Phase 3: Sync Layer (2 to 3 weeks)

In bitsocial-react-hooks, so every client gets it.

- [ ] A store of nearby publications, each verified with pkc-js before it is kept.
- [ ] A queue of the user's own posts written while offline.
- [ ] On reconnect, sign each queued post again with a fresh timestamp and publish it.
- [ ] Queued, publishing, accepted and rejected states.
- [ ] Supply challenge answers up front where the community allows it.
- [ ] Decide what happens to replies and votes on nearby-only posts, and implement it.

**Check:** a post written with no internet shows on a second phone nearby, then appears in its community after the author's phone reconnects.

## Phase 4: Nearby in One Client (2 to 3 weeks)

Proposed first client: Seedit, since nearby posts carry text and links only. Not decided.

- [ ] Experimental settings toggle, off by default.
- [ ] Nearby view, labelled as not accepted by any community yet.
- [ ] Block by author, and a way to hide the Nearby view.
- [ ] Decide whether Nearby uses a separate account by default.
- [ ] Field test with a handful of phones in one place.
- [ ] Ship in an experimental Android release.

**Check:** people outside the team can turn it on and exchange posts without help.

## Phase 5: More Clients

- [ ] Add the setting to 5chan.
- [ ] Add it to the profile-based client once that exists. This is the best fit (see [how-it-could-work.md](how-it-could-work.md#fit-by-client)).
- [ ] Check whether Electron can take part on desktop.

## Later: Longer Range

Not scheduled and not estimated. LoRa, gateways and mesh-hosted communities wait on the sizes from Phase 0 and on a decision about delayed publications. See [open-questions.md](open-questions.md#later-longer-range).
