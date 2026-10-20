# Spark Program | Bees Network — Decentralized Observation Layer for AI Agents on CKB

*Category: **DAOs & Funding → Spark**. Applicant: North Bear (NOrththebear) · Discord: northbears · GitHub: github.com/NOrththebear*

## 1. Project Overview / 项目概述

- **Project name:** Bees Network
- **One-line summary:** A desktop "bee" node that lets anyone contribute real-world observations (HTTP, DNS, TLS) to AI agents and get paid on-chain in CKB for verified work.
- **Positioning:** Developer tool + end-user desktop application + on-chain reward protocol (a working MVP already deployed on CKB **Pudge testnet**).

Bees Network turns ordinary computers into a distributed observation layer for AI agents. Each node runs a small desktop app, receives observation tasks from an orchestrator, executes them, reports results, and is paid per epoch through CKB type/lock scripts. No custody, no manual claiming, no operator key on the user's machine.

## 2. Team Profile / 团队简介

- **Applicant / core developer:** North Bear (NOrththebear) — full-stack + Rust, protocol and contract development.
- **GitHub:** https://github.com/NOrththebear (project repo: https://github.com/NOrththebear/Bees-Network — public)
- **Email:** poisapink@gmail.com
- **Discord:** northbears
- **Telegram:** —
- **Background:** Built the full MVP end-to-end: Tauri/Next.js desktop client, Rust CKB integration (ckb-sdk, molecule codecs, secp256k1 sighash), three on-chain contracts (registry, epoch, rewards-lock), the orchestrator/coordinator services, and the hosted testnet deployment. Comfortable across the CKB Cell Model, type-script validation, and transaction assembly.

## 3. Project Background / 项目背景

AI agents increasingly need fresh, real-world observations — "what does this endpoint actually return", "is this DNS record live", "is this TLS certificate valid from where I am". Today that work is either done by a single centralized fetcher (a single point of failure and a single point of trust) or is economically unattractive for individuals to provide.

Concretely: an agent framework wants to verify a service health check from many network vantage points. There is no simple way for a person with a spare machine to say "I will serve these observations and be paid for the ones I actually delivered". Existing solutions are either centralized APIs or general compute marketplaces that are far too heavy for a 50 ms HTTP probe.

Relevance to CKB: the reward accounting is a natural fit for the Cell Model. Each epoch is a cell, each payout is a share cell locked by a merkle-gated lock script, and the registry is an operator-gated singleton. CKB gives us cheap, auditable, non-custodial settlement for micro-work — something a database + Stripe backend cannot do without trust.

### Category reference: Grass (https://www.grass.io/)

Grass is the strongest proof that this category is real. It is a desktop/mobile/browser-extension client that lets **8.5M+ people share unused internet bandwidth**; verified institutions consume that bandwidth to reach live web data for real-world and AI applications, and contributors earn Grass Points that convert to GRASS tokens or USDC. In other words, Grass already convinced millions of ordinary users to run a lightweight background node as part of an AI data layer.

That validates the three assumptions Bees is built on: (1) people will run a small client for an AI data layer; (2) demand for fresh, real-world web observations is real and paid for; (3) a downloadable desktop node is a viable distribution channel.

Bees is deliberately adjacent, not a clone:

| | Grass | Bees Network |
|---|---|---|
| Unit of work | shared bandwidth / proxied traffic | an explicit, typed observation task (HTTP GET/POST, DNS, TLS) |
| Accounting | off-chain points, periodic token/USDC distribution | on-chain epoch + share cells, merkle-committed each epoch |
| Settlement | token/USDC distribution | native CKB, non-custodial, claimed from the app |
| Buyer | institutions buying bandwidth/data access | AI agents and developers buying verifiable observations |
| Trust model | institution-gated access | wallet-attested nodes, on-chain registry, per-epoch audit |

Grass proves the demand and the distribution; Bees supplies the missing primitive — **verifiable, individually settled observations that an agent can call directly**, with the accounting itself living on CKB.

## 4. Solution / 解决方案

**Core approach.** A user downloads the app, the app generates/links a wallet, and the node registers with the network. The orchestrator hands out small observation tasks; the node executes them and reports results ("beats"). A coordinator periodically closes an epoch: it reads the live tally, computes each node's points, and mints one epoch cell plus one reward share cell per earner. The node claims its share directly from the dashboard (auto-claim is on by default).

**User perspective.** Download → open → start earning. No funds, no keys from us, no configuration. The dashboard shows live beats, bytes served, epoch history, and claimable CKB.

**Differentiation.** (1) It is an *observation* network, not a general compute market — tasks are tiny and latency-bound, so a laptop is enough. (2) Payouts are on-chain and auditable (epoch + share cells), not an internal ledger. (3) The desktop client holds no operator secrets; registration and epoch closing are server-side protocol roles, so a compromised client cannot mint. (4) Against **Grass** (the category's benchmark), Bees is not a bandwidth resale marketplace: it sells *verifiable observation results*, settles every epoch natively on CKB, and needs no speculative token to pay contributors.

## 5. Technical Approach / 技术方案

- **Client:** Tauri v2 + Rust, Next.js/React frontend, system keyring for wallet keys, Linux `.deb`/AppImage today.
- **Chain integration:** Rust `ckb-sdk`, hand-audited molecule encoders (registry / epoch / share), `secp256k1_blake160_sighash_all`, data2 type scripts.
- **Contracts:**
  - `registry-type`: operator-gated singleton; allows exactly one append per transition (Sybil/double-write protection).
  - `epoch-type`: mints one epoch cell per id, commits the points merkle root and rewards root.
  - `rewards-lock`: releases a share only to the wallet named in the epoch's rewards ledger.
- **Services:** an HTTP orchestrator (task queue, tally, `/stats`) and a coordinator loop that closes epochs, deployed on a hosted testnet environment; both already live.
- **Key challenges (and current solutions):**
  1. *Molecule correctness* — hand-rolled serializers drifted; solved by reference-encoding with `ckb-cli` and byte-exact tests.
  2. *Fee/capacity edge cases* — sub-minimum change must be folded, not emitted (fixed); registry appends need a minimum fee (fixed).
  3. *Key custody* — operator signing moved entirely server-side; the client never sees it (fixed in the distribution build).
  4. *Remaining (this grant):* epoch cells are currently burn-proof (300 CKB locked per epoch), which is economically unsustainable; the coordinator is stateful/restart-fragile; there is no Sybil/task-validation hardening yet.

## 6. Execution Plan / 执行计划

**8 weeks.**

- **Week 1 — Protocol economics hardening.** Redesign epoch-cell lifecycle so closed epochs are reclaimable (e.g., maturity window + operator path) instead of permanently burning 300 CKB; fix change/fee handling; add regression tests on a local devnet. *Milestone: epoch cycle no longer leaks CKB.*
- **Week 2 — Service resilience.** Persist orchestrator tally + coordinator state; make the epoch loop restart-safe and idempotent; add health/metrics endpoints. *Milestone: restart the services without losing nodes or double-closing.*
- **Week 3 — Security & abuse resistance.** Authenticate admin endpoints, rate-limit, add wallet-signature attestation for node registration, and define eligibility/task-validation rules; publish a short threat model. *Milestone: threat model + mitigations shipped.*
- **Week 4 — Cross-platform distribution.** Windows `.msi` and macOS `.dmg` builds, code signing, and an auto-update feed; keep Linux polished. *Milestone: signed installers for all three OSes.*
- **Week 5 — Beta onboarding.** In-app onboarding, diagnostics/error reporting, one-click claim, guided first-run; ship `v0.2.0-beta`. *Milestone: beta build published.*
- **Weeks 6–7 — Beta distribution & user testing.** Recruit 20–30 testers; run structured test scripts (install, register, earn, claim, restart); privacy-preserving telemetry; weekly feedback synthesis and fixes for top issues. *Milestone: beta test report with metrics.*
- **Week 8 — Close-out.** Final `v0.2.0` release, demo video, docs, completion report, public results post.

## 7. Required Funding & Breakdown / 所需资金与资金分配明细

**Requested:** **USD 2,000** — payable in **CKB + USDI** (USDI ≤ 50%).

**Why > $1,000 (required justification).** This is not a single-category request. It combines (a) protocol-level work on on-chain contracts and coordinator economics — changing the epoch lifecycle and hardening security — with (b) a real distribution/beta program that recruits and supports outside users. The two are coupled: the beta cannot run safely until the economics and service resilience are fixed, and the fixes cannot be validated without real users. A $1,000 budget funds roughly one of those halves, not the delivered whole.

| Item | Amount | Use |
|---|---|---|
| Protocol & economics hardening (W1–W2) | $700 | Epoch reclaimability, change/fee correctness, restart-safe services, tests, CI |
| Security & abuse resistance (W3) | $300 | Admin auth, rate limiting, wallet attestation, eligibility, threat model |
| Cross-platform packaging & signing (W4–W5) | $400 | Windows/macOS builds, signing, auto-update feed, beta build |
| Beta distribution & user testing (W6–W7) | $400 | Tester recruitment, support, telemetry, feedback synthesis, fixes |
| Docs, demo video, completion report (W8) | $150 | Documentation, demo, reporting |
| Test infra / contingency | $50 | CI runners, hosted testnet, test tokens |
| **Total** | **$2,000** | |

## 8. Deliverables + Verification / 交付物 + 验证方式

**Deliverables (with acceptance criteria).**
1. **Open-source repo** at `v0.2.0` (existing repo continues) — builds from a clean checkout; license present.
2. **Signed installers** for Linux, Windows, macOS + a download page — all install and launch.
3. **Test suite + CI** — `cargo test` and frontend tests pass in CI on every push.
4. **On-chain evidence on Pudge testnet** — explorer links for a registry append, an epoch close, and a share claim.
5. **Beta report** — N≥20 testers, tasks completed, issues found/fixed, retention, verbatim feedback.
6. **Demo video + docs** — architecture, run instructions, "verify in 5 minutes" guide.
7. **Completion report** — plan vs actual, fund usage, lessons.

**How to verify without code review (key requirement).**
- Download an installer, run it, complete onboarding, and watch beats appear on the dashboard / `/stats` (compare with the live-MVP dashboard screenshot in §9).
- Open the provided explorer links and confirm the registry, epoch, and claim transactions on-chain.
- Run the published test commands (or open the CI run links) and see them pass.
- Read the beta report and watch the demo video.

**Expected output:** a running node earning testnet CKB and claimable shares, plus green CI and on-chain tx hashes.
**Environment:** any desktop Linux/Windows/macOS with internet; no node software or funds required to verify.
**Verification cost:** ~5 minutes per item; no committee code review needed.

## 9. Current State vs Funded Work / 当前状态与资助范围

**Already done (unfunded MVP).**
- Working Tauri desktop node + Next.js dashboard; Rust CKB integration.
- Three contracts deployed on Pudge testnet; registry, epoch, and rewards-lock validated on-chain.
- A complete reward cycle has run: epochs closed, share cells minted, and shares claimed from the app.
- Hosted orchestrator + coordinator; multi-node registry appends; server-side registration so the client holds no operator key.
- Public Linux release `v0.1.0` (`.deb` + `.AppImage`).

![Bee Network Node dashboard — the live MVP on Pudge testnet](https://raw.githubusercontent.com/NOrththebear/spark-bees-network/main/assets/mvp-dashboard.png)

*The existing MVP, running live: node active and eligible, epoch 497638, 1.66K earned credits / 146 observation points, 4,106 requests served and 0.29 MB transferred.*

**Funded by this Spark grant (the delta).**
- Epoch economics fixed so the protocol stops permanently burning CKB per epoch.
- Restart-safe, observable services; security/abuse hardening; documented threat model.
- Signed cross-platform installers and auto-update.
- A real beta with 20–30 external users, measured and reported.

## 10. CKB Alignment / CKB 契合度

- **Cell Model used for real accounting:** the registry is a singleton cell whose type script enforces one append per transition; each epoch is a cell carrying the points merkle root and rewards root; each payout is a cell locked by a merkle-gated `rewards-lock` script. This is not a token wrapper — the protocol's state machine *is* cells and scripts.
- **Native settlement:** contributors are paid in CKB directly, with no custody and no off-chain ledger as the source of truth.
- **Auditability:** every epoch and claim is independently checkable on the explorer.
- **Honest scope note:** AI observation tasks themselves run off-chain (they are network probes); CKB provides identity, registry, accounting, and settlement. Planned follow-ups include `did:ckb`/JoyID login so users can bring an existing CKB identity instead of a generated key.

---

### Identity / repo confirmation
- Applicant GitHub: https://github.com/NOrththebear
- Project repo: https://github.com/NOrththebear/Bees-Network (public)
- Spark identity repo (per Jul 2026 guidance): https://github.com/NOrththebear/spark-bees-network
