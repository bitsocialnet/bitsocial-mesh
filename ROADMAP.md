# Roadmap

**Status: sizes measured, nothing built.** Nothing below is scheduled.

The goal is that Bitsocial communities keep working with no internet. The phases follow the stages in [how-it-could-work.md](how-it-could-work.md): a community box first, then radio links between boxes. Delayed delivery and phone to phone are kept as later options.

The durations are rough guesses for one developer working full time. The first phase that meets the goal is Phase 2. A first guess for reaching the end of it is 1 to 2 months, with low confidence until Phase 1 shows what breaks offline.

Each phase ends with a check. If the check fails, the work stops there and the reason is written into [open-questions.md](open-questions.md).

Phase 1 needs no special hardware and can start at any time.

## Phase 0: Measure (done)

- [x] Measure the encoded size of a signed post, reply and vote as pkc-js produces them. A typical post or reply is 550 to 850 bytes, a vote 400 to 480.
- [x] Measure a community's latest signed state and one page of posts. Settings are about 5 KB; the whole record is 79 to 202 KB compressed.
- [x] Check whether bitchat phones relay packet types they do not recognise. Both apps do, from reading the source. Not tested on phones.
- [x] Confirm bitchat-android's licence terms. GPL version 3 text, with nothing saying "only" or "or later", and a README that still says public domain.

**Check:** passed on 2026-11-14. The sizes are in [how-it-could-work.md](how-it-could-work.md#measured-sizes). They matter for Phases 3 and 5; over Wi-Fi nothing needs shrinking.

## Phase 1: Blackout Test on a Local Network (2 to 4 days)

Needs two computers and a phone on one Wi-Fi network with no internet behind it.

- [ ] Host a community on one machine and cut the internet.
- [ ] From a second machine on the same network, load the community, post, reply and vote.
- [ ] Serve the web client from the host and open it in a phone's browser. Note what the browser refuses to do on a local address without HTTPS.
- [ ] Write down everything that fails or stalls: name resolution, finding peers, challenges, timeouts.
- [ ] Try each of the common challenges offline.
- [ ] Reconnect the internet and confirm that what was posted offline appears on the wider network unchanged.

**Check:** there is a list of what stops a community from working on an isolated network, and each item is marked as a setting, a client change or a protocol change. This decides how large Phase 2 is.

## Phase 2: The Community Box (not estimated until Phase 1; first guess 3 to 6 weeks)

The first phase that meets the goal.

- [ ] Fix or work around what Phase 1 found.
- [ ] A way for browsers on the hotspot to read and post without being able to manage the community.
- [ ] A blackout mode in the desktop app: open a hotspot, host the user's communities on it, serve the client.
- [ ] Create a new community on the box with no internet and use it from phones.
- [ ] A set of challenges that work offline.
- [ ] Test with a handful of phones, including an iPhone, and measure how many one hotspot can serve.
- [ ] A ready-made image for a small computer such as a Raspberry Pi.
- [ ] Copy a box's data to a second machine that has internet and confirm the community publishes from there unchanged.

**Check:** with no internet anywhere, people who never installed anything join a laptop's hotspot, open the community in a browser, post and reply, and a moderator removes a post. When the laptop gets a link out, the community and everything in it appear on the wider network.

## Phase 3: Linking Boxes (not estimated)

Needs two LoRa radios and two boxes.

- [ ] Decide the transport. Reticulum is the working assumption.
- [ ] Carry the publishing exchange between a client on one box and a community on another, and time it. It has to finish well inside 5 minutes.
- [ ] A compact way to send new posts and updates between boxes.
- [ ] A box with internet relays for boxes without it.
- [ ] Check the radio rules for the first target region.

**Check:** someone on one box's hotspot posts to a community hosted on a second box a kilometre away, with no internet at either.

## Phase 4: Delayed Delivery (on demand; not estimated)

Not started unless people using boxes ask for it. It needs a protocol change, decided by the pkc-js maintainers, and it costs more than it first appears (see [how-it-could-work.md](how-it-could-work.md#stage-3-delayed-delivery)).

- [ ] Agree the design with the pkc-js maintainers: opt-in per community, posts and replies only, received time recorded by the host, delayed posts sent to the approval queue.
- [ ] Implement it in pkc-js.
- [ ] In the clients: hold other people's signed posts, and publish them when a link appears.

**Check:** a post signed with no internet is carried out by a different phone hours later and reaches the approval queue of a community hosted elsewhere.

## Phase 5: Phone to Phone (optional; about 2 to 3 months if built)

Decide first whether it is worth building (see [open-questions.md](open-questions.md#is-it-worth-building)). Needs three or more physical Android phones.

- [ ] Decide the wire format and whether to reuse bitchat-android's Bluetooth code. Proposals in [open-questions.md](open-questions.md#wire-format).
- [ ] Capacitor plugin: two phones exchange one signed post with screens off. Measure battery and speed.
- [ ] The mesh: fragmentation, hop limit, duplicate suppression, neighbour sync, retention.
- [ ] Sync layer in bitsocial-react-hooks: a store of nearby posts, a queue of the user's own, publishing on reconnect.
- [ ] A Nearby view in one client, labelled as not accepted by any community.

**Check:** a post written on phone A reaches phone C through phone B, and appears in its community after the author's phone reconnects.

## Later

Not scheduled and not estimated.

- **More clients:** the same support in 5chan, Seedit and the profile-based client once that exists.
- **A map** of communities by coarse location, online only and opt-in. See [open-questions.md](open-questions.md#a-map).
