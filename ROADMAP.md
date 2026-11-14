# Roadmap

**Status: Phase 0 measured, two decisions open.** Nothing after Phase 0 is scheduled.

The goal is that Bitsocial communities keep working with no internet. The first phase that shows this is Phase 5. Phases 1 to 4 build what it stands on.

The durations are rough guesses for one developer working full time. Phases 1 to 4 come to about 3 to 4 months. Phase 5 can't be estimated until Phase 1 shows what breaks offline; a first guess for reaching the end of it is 4 to 6 months in total. Phase 6 is protocol work that depends on the pkc-js maintainers and has no estimate.

Each phase ends with a check. If the check fails, the work stops there and the reason is written into [open-questions.md](open-questions.md).

Phases 1 and 6 need no phones and can start at any time.

## Phase 0: Measure and Decide (1 to 2 days)

- [x] Measure the encoded size of a signed post, reply and vote as pkc-js produces them. A typical post or reply is 550 to 850 bytes, a vote 400 to 480.
- [x] Measure a community's latest signed state and one page of posts. Settings are about 5 KB; the whole record is 79 to 202 KB compressed.
- [x] Check whether bitchat phones relay packet types they do not recognise. Both apps do, from reading the source. Not tested on phones.
- [x] Confirm bitchat-android's licence terms. GPL version 3 text, with nothing saying "only" or "or later", and a README that still says public domain.
- [ ] Decide the wire format: an own Bluetooth service and packet format, or bitchat's. Proposal in [open-questions.md](open-questions.md#wire-format).
- [ ] Decide whether to reuse bitchat-android's Bluetooth code, port the iOS code, or write from the whitepaper. Proposal in the same place.

**Check:** passed on 2026-11-14. The sizes are in [how-it-could-work.md](how-it-could-work.md#measured-sizes): a typical post needs 2 Bluetooth fragments and a vote fits in one frame. The two decisions are still open.

## Phase 1: Blackout Test on a Local Network (2 to 4 days)

Needs two computers on one Wi-Fi network, or one computer with an isolated test network. No phones, no Bluetooth.

- [ ] Host a community on one machine and cut the internet.
- [ ] From a second machine on the same network, load the community, post, reply and vote.
- [ ] Write down everything that fails or stalls: name resolution, finding peers, challenges, timeouts.
- [ ] Reconnect the internet and confirm that what was posted offline appears on the wider network unchanged.
- [ ] Try each of the common challenges offline.

**Check:** there is a list of what stops a community from working on an isolated network, and each item is marked as a setting, a client change or a protocol change. This decides how large Phase 5 is.

## Phase 2: Two Phones (1 to 2 weeks)

Needs two physical Android phones, ideally from different makers. Emulators are not known to support this.

- [ ] Capacitor plugin skeleton: advertise, scan, and act as both central and peripheral.
- [ ] Send one signed post from phone A to phone B and verify it with pkc-js in the app.
- [ ] Keep it running with the screen off through a foreground service.
- [ ] Measure battery use and transfer speed over several hours on both phones.

**Check (go or no-go):** posts cross reliably with both screens off, and the battery cost is one a user would accept. This is the largest technical risk in the project.

## Phase 3: The Mesh (3 to 4 weeks)

Needs three or more phones.

- [ ] Fragmentation and reassembly for publications larger than one packet.
- [ ] Hop limit, duplicate suppression and a random delay before relaying.
- [ ] Periodic sync between neighbours of what each phone holds.
- [ ] Retention: decide and implement how long other people's publications are kept and re-shared.
- [ ] Per-connection rate limits.

**Check:** a post written on phone A reaches phone C through phone B, and a phone carried from one group to another delivers what it holds.

## Phase 4: Sync Layer and the Nearby Feed (4 to 6 weeks)

The part that needs no protocol change. It is a first release, not the goal.

In bitsocial-react-hooks:

- [ ] A store of nearby publications, each verified with pkc-js before it is kept.
- [ ] A queue of the user's own posts written while offline.
- [ ] On reconnect, sign each queued post again with a fresh timestamp and publish it.
- [ ] Queued, publishing, accepted and rejected states.

In one client (proposed: Seedit, since posts on the mesh carry text and links only; not decided):

- [ ] Experimental settings toggle, off by default.
- [ ] Nearby view, labelled as not accepted by any community yet, with blocking by author.
- [ ] Decide whether the mesh uses a separate account by default.
- [ ] Ship in an experimental Android release.

**Check:** people outside the team can turn it on and exchange posts without help, and a post written with no internet appears in its community after the author's phone reconnects.

## Phase 5: A Community on the Mesh (not estimated)

The first phase that meets the goal. Its size depends on Phase 1.

- [ ] A bridge beside the host that carries the publishing exchange between mesh packets and the community's node.
- [ ] Time the exchange across one, two and three hops. It has to finish well inside 5 minutes.
- [ ] Send accepted posts to phones one at a time, each with the host's signed update.
- [ ] Announce hosted communities on the mesh, and show them in the client.
- [ ] Create a new community on a laptop with no internet and use it from phones.
- [ ] Fix or work around what Phase 1 found.

**Check:** with no internet anywhere, a phone posts to a community hosted on a laptop two hops away, a second phone replies, and a moderator removes a post. When the laptop gets a link out, the community and everything in it appear on the wider network.

## Phase 6: Delayed Delivery (not estimated)

Protocol work, decided by the pkc-js maintainers.

- [ ] Write the proposal as a BSIP: a community opts in to a longer window for delayed publications.
- [ ] Answer the open points in [open-questions.md](open-questions.md#delayed-delivery): backdating, who runs the exchange, how the author learns the result, duplicates.
- [ ] If accepted, implement it in pkc-js and in the sync layer.

**Check:** a post signed with no internet is carried out by a different phone hours later and accepted by a community hosted elsewhere.

## Phase 7: More Clients

- [ ] Add the setting to 5chan.
- [ ] Add it to the profile-based client once that exists (see [how-it-could-work.md](how-it-could-work.md#fit-by-client)).
- [ ] Check whether the desktop app can join the Bluetooth mesh directly.
- [ ] Let the Android app pass its install file to a nearby phone.

## Later

Not scheduled and not estimated.

- **Longer range:** LoRa, gateways and Reticulum. See [open-questions.md](open-questions.md#later-longer-range).
- **A map** of communities by coarse location, online only and opt-in. See [open-questions.md](open-questions.md#a-map).
