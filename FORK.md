# Amperstrand iso14443-rs — experimental fork

This is our **experiment and integration fork** of
[Foundation-Devices/iso14443-rs](https://github.com/Foundation-Devices/iso14443-rs).

- **Not a replacement** for upstream. We track upstream and rebase/port onto
  it; `main` (upstream) is the source of truth.
- **Playground purpose**: `ai-experiments` is where we test quickly, A/B
  alternatives, and run our own projects (bolty-rs, ccid-firmware-rs pin this
  branch) before anything is proven.
- **Upstream policy**: once an experiment is fully working, it is ported to
  current upstream `main` and proposed as a **small, clean PR** — see the
  table below for per-item plans and triggers. First result of this policy:
  PR #8 (no_std FWT/SFGT) is upstreamed from this fork.
- **License**: GPL-3.0-or-later, inherited and documented in `LICENSING.md`.

## Experiments on this branch

| Experiment | Commit | Why it exists / what it tests | Status | Upstream plan |
|---|---|---|---|---|
| **SFT-7605 port: strict CRC_A** | ~~271ade9~~ *(archived — superseded by the 73db6d4 rebase: upstream's SFT fix set now carries the substance)* | Every parser rejected forged `00 00` placeholder CRCs (same class upstream fixed in 6e2e485); hw-verified paths now re-stamp with the computed CRC; `Block::to_vec` emits real CRC | **Landed on fork** (tests: forged rejected, genuine round-trips) | Upstream→fork direction; nothing to send back |
| **SFT-7607 port: peer work bounds** | ~~15d8a03~~ *(archived — superseded by the rebase)* | WTX flood looped forever; chain accumulation unbounded under alloc. Now: 8-consecutive-WTX budget + MAX_CHAIN_LEN (4096/1024), mirroring upstream defaults | **Landed on fork** (flood + over-budget tests) | Upstream→fork direction |
| **SFT-7606 audit: ATS short inputs** | ~~eab23f6~~ *(archived — superseded by the rebase)* | Verified the fork's ATS parser already bounds-checks TL and interface bytes (no underflow class); pinned with 7 adversarial cases | **Verified, pinned** | Nothing needed |
| **PR #8 mirror: no_std fwt()/sfgt()** | ~~6399d5a~~ *(archived — superseded by the rebase; PR #8 remains open upstream)* | Our own upstream proposal adopted on the playground; no_std drivers compute card-declared FWT directly | Landed on fork (fwt/sfgt tests now run in every cell) | **PR #8 open upstream** — same change |
| Borrow-free `PcdSession` | 3b453d5 | Splits protocol state from the transceiver so drivers keep hardware ownership without borrow gymnastics; enables our MFRC522 re-entrant driver | In use (ccid-firmware-rs) | Only if maintainers want the architecture; otherwise stays fork-local as an adapter |
| S(WTX) timeout extension | 82f706e + 15d8a03 | NTAG424 DNA sends S(WTX)/WTXM; PCD must extend FWT instead of timing out. `try_set_timeout_ms` hook + `WTXM × base_fwt` computation, now with flood budget | Proven on hardware in our projects | **Issue drafted** (`wtx-timeout-issue-draft.md`): upstream has no timeout surface at all — API-design conversation first; post after PR #8. Fork = reference implementation |
| R(ACK) block-number toggle | c911904 | Correctness fix: toggle PCB block number before building R(ACK) in chaining recovery | Proven here | **VERIFIED vs upstream: already fixed upstream** (role-based handler, 7a4f134). Nothing to port |
| Chain recovery re-send | 228714a | Re-sends the original block during chain recovery instead of a stale one | Proven here | **VERIFIED vs upstream: self-consistent** (rebuilds with no intervening toggle). Nothing to port |
| Pinned deps / lint / fmt | 945bfec, 268db79 | Reproducible builds for our CI lanes | Standing config | Fork-local, not proposed |
| `LICENSING.md` | d3b00b8 (fork) / f07fe42 (upstream branch `licensing-faq`) | Fork: GPL inheritance chain. Upstream-adapted: plain-language licensing FAQ | Upstream branch **prepared** | Ready to PR: patch `fork-align/0000-upstream-licensing-faq.patch`, body `licensing-pr-body.md` — owner posts when ready |
| Diverged baseline | (history) | Fork branched from upstream at `c9834fa`; now **11 ahead / 24 behind**. Upstream's unique side fully triaged this session: SFT-7605/07 ported, SFT-7606 verified absent, role-based handler + limits.rs architecture consciously not merged (keeps PcdSession) | Maintained | Never proposed wholesale; port behaviors only. No remaining known-needed upstream items |

Ground rule: every new experiment adds or updates a row here — what it is,
why it is being tested, and the trigger for considering it upstream
(typically: "proven in ≥1 shipped project + upstream main still lacks it").
