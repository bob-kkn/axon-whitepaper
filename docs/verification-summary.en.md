# Axon Design Validation Summary

> A record of validating the whitepaper's core claims through eight independent prototypes — five validation slices and three integration slices. For investor and partner review.
> All results are pre-testnet prototype output; confirmation is left to testnet measurement.

## In brief

The Axon whitepaper makes three large claims — that the **issuance economics** of compute supply are safe against fraud, that the **verification** (re-execution adjudication) protecting that issuance is **actually feasible**, and that the **payment and verification plumbing** built on top **withstands attack**. Within the whitepaper these three axes are arguments, but we did not leave them as assertions. Five independent prototypes each realized a claim in code and stress-tested it under adversarial conditions, and all three axes passed a first-pass validation. Three further slices then took the **abstractions** those validations had left behind — an in-memory ledger, an abstract randomness source, an abstract adjudication oracle — and replaced each with a real structure, to ask whether the assumptions still hold on real plumbing, and what each assumption actually costs. What matters is that these prototypes hold each other up: one slice's measurement fills another slice's assumption, and yet another slice confirms that the assumption holds on the real plumbing. Below is an honest summary of what we verified, what held, and what still remains the province of the testnet.

## What we verified — the five validation slices

| Slice | Question verified | Method | Key result | Honest limit |
|-------|-------------------|--------|------------|--------------|
| **A. Issuance economics (`sim/`)** | Is the expected value of fraudulent issuance really negative at the draft parameters | Agent-based Monte Carlo + closed-form oracle regression | Under four attacks (adaptive, distributed, collusion, evasion) the ch7 defaults keep EV<0. **The center of gravity of the defense is detection, not the deposit** — cumulative detection converges to 1, so it holds even with the deposit driven extremely low | Four conservative model simplifications disclosed. The sole collapse point is full verifier collusion + disappearance of third-party re-checks |
| **B2. Re-execution adjudication (`verify/`)** | Is the re-execution adjudication that A *assumed* feasible actually feasible with real LLMs | Real-model experiment (Qwen2.5, SmolLM2, heterogeneous ollama) | Homogeneous runtimes adjudicate perfectly by regeneration (AUC 1.0), complementary with teacher-forcing. **Heterogeneous runtimes adjudicate imperfectly** — worst-case false-negative rate (FAR) ≈ 0.375 | A's "per-sample detection = 1" holds only for homogeneous runtimes; heterogeneity was an optimistic assumption, shown by measurement |
| **Feedback (`sim/` FAR)** | Does the economy hold once the imperfection of heterogeneous adjudication is injected | Feed B2's measured FAR (0.25–0.375) back into the A engine and re-sweep | The ch7 defaults keep EV<0 across **all 96/96 scenarios; no deposit increase needed.** Even as per-epoch adjudication leaks, the 24-epoch cumulative detection stays ≥0.997 | Adjudication imperfection modeled as a single parameter (correlated failure is a separate task) |
| **C. Payment channel (`channel/`)** | Is the honest party protected under exit fraud and offline exit in an L2 channel | Real ed25519 signatures + commit-close-dispute state machine | Latest-signed-state-wins dispute preserves the honest party's share under both attacks. **Four invariants hold over 500 arbitrary histories with 0 violations** | Genesis, group channels, and real networking are out of scope |
| **B1. PoI plumbing (`poi/`)** | Does the commit-sample-slash plumbing of issuance verification withstand protocol attack | Real sha256 Merkle + public-randomness sampling + slashing | All four attacks (tamper, non-disclosure, inclusion forgery, grinding) resolve to slashing. Four invariants over 300+ epochs, 0 violations. **The measured detection rate reproduces A's oracle `1−(1−f)^k` within 3σ** | The actual ledger derivation of the public randomness and the re-execution adjudication are separate layers (replaced by abstractions) |

## Stripping out the abstractions — the three integration slices

The five validation slices left abstractions in three places: the L1 ledger was an in-memory dictionary (`MockL1`), the sampling randomness was a premise ("unpredictable after commit"), and the re-execution adjudication was a true/false oracle. Three integration slices replaced each with a real structure.

| Slice | Abstraction removed | Method | Key result | Honest limit |
|-------|---------------------|--------|------------|--------------|
| **D1. DAG L1 ledger (`ledger/`)** | `MockL1` → a real block-less DAG | Account-based + DAG (parent references), five Axon tx types (deposit, settle, issue, slash, transfer) | In a double-spend conflict **exactly one confirms, deterministically** (single-state fold). Only issuance grows the supply, split exactly into supplier share + verification tax. **Six invariants hold over 300+ random transaction DAGs with 0 violations** | P2P networking, probabilistic finality, and transaction signature authentication are out of scope |
| **D2. Randomness beacon (`beacon/`)** | Abstract seed → a real commit-reveal beacon | sha256 commit-reveal + exact grinding enumeration | Achieves unpredictability and public verifiability. But **a last revealer can select among 2^k outputs** → detection degrades from `d` to `d^(2^g)` (0.958→0.841 at g=2). Withhold penalties bound g and accumulation recovers it | A VDF to remove the residual bias is roadmap (we did not build a toy VDF) |
| **D3. PoI×adjudication wiring (`d3/`)** | Abstract `is_forged` → adjudication wired into the real poi protocol | Import poi for real and inject a B2-measured-FAR adjudicator into verdict's adjudicator slot | The integrated detection `1−(1−f(1−FAR))^k` emerges from the real pipeline (at FAR=0 it reduces to B1). **Four composition invariants over 300 epochs, 0 violations** | Adjudication is a deterministic stub reflecting B2's measured FAR (real LLM re-execution is B2's province) |

## How it all connects — the chain of validation

The eight slices run independently, but they pass assumptions and measurements back and forth to form a single chain.

- **A assumed two things** — that the detection probability of sampling is `1−(1−f)^k` (the sampling oracle), and that re-execution adjudication is perfect.
- **B1 realized the first assumption.** When we built the plumbing with real Merkle commitments and post-commit public-randomness sampling, the measured detection rate over many epochs reproduced exactly the closed form A had premised. **A's economic conclusion stands not on an abstraction but on the real plumbing.**
- **B2 refuted the second assumption.** In real-LLM re-execution, heterogeneous-runtime adjudication was not perfect (FAR 0.25–0.375). We injected that measured value back into A as **feedback** and re-swept — and even with imperfect adjudication, repetition and accumulation catch it downstream, so the economic defense still held.
- **C realized the payment path (Chapter 6), and B1 the issuance path (Chapter 7)**, showing by invariants that the dispute and slashing rules the whitepaper had deferred to "fixed at the implementation stage" actually protect the honest party.
- **D1, D2, and D3 stripped out the remaining abstractions.** D1 put a real DAG where `MockL1` had stood, showing double-spend prevention and value conservation by invariants; D2 built the randomness B1 had premised as "unbiasable" as a real beacon and quantified **what that premise costs** (last-revealer grinding); and D3 wired B2's measured-FAR adjudication into B1's abstract adjudication slot to confirm the two slices compose correctly.
- **And D3 refined the feedback in turn.** The feedback sweep rolled adjudication failure once per epoch, but the real protocol re-adjudicates each sampled job independently. So the real detection rate is **higher** than the value the feedback used — meaning A's economic conclusion was a conservative estimate that understated detection relative to the real wiring.

In short, B2 measures reality and feeds A, B1 realizes A's sampling mechanism and confirms A, C and B1 guard the plumbing of both paths, and D1, D2, and D3 replace the abstractions beneath that plumbing with real structures while presenting the bill for each assumption.

## The four most important findings

1. **Security depends on detection, not on the size of the deposit.** The intuition is "grow the deposit and you are safe," but the simulation pointed to a different center of gravity. Issuance is a repeated game, so independent sampling repeats each epoch and the cumulative detection probability of a node that continues forging converges to 1. Thus the expected value stayed negative even with the deposit driven extremely low, and the only point where the defense actually collapsed was when verifiers all colluded and third-party re-checks vanished. **What bears the load of the design is not the collateral but the random assignment of verifiers and the openness of challenge.**

2. **Heterogeneous-runtime adjudication is imperfect, but the economy holds anyway.** Reproducing the same inference across different hardware and runtimes is hard because of floating-point nondeterminism, and a measured false-negative rate existed (worst-case ≈0.375). This refutes the optimistic assumption but is not a catastrophe — a single epoch's adjudication failure is caught in later epochs, the per-trial cumulative detection exceeds 0.997, and once caught in the end, deposit forfeiture overwhelms the gain. **Even with the measured condition injected, the economic defense of the issuance path holds.**

3. **Every time we made an abstraction real, the assumption presented a bill — and both bills were payable.** When we built the sampling randomness the whitepaper had premised as "unbiasable" as a real commit-reveal beacon, the participant who reveals last could choose among a bounded set of outcomes by deciding whether to withhold its own commitment — detection drops from `d` to `d^(2^g)`. In exchange, a withheld commitment is itself detectable and slashable, so that choice carries a price, and the accumulation of the repeated game recovers the loss. There was also a bill in the opposite direction: the fact that the real protocol re-adjudicates **each** sampled job **raises** detection above the coarse approximation the economic simulation used. **Making an assumption real moves some numbers the wrong way and some the right way — we wrote down both.**

4. **Payment and plumbing security were verified by invariants and adversarial scenarios, and review caught real holes.** The security properties of the eight slices were not merely passed through a few scenarios but verified as **invariants** over hundreds to thousands of arbitrary histories, epochs, and transaction DAGs. Each prototype went through an adversarial review before merge, and the review caught real defects — a cooperative close replaying a stale state (fixed by domain separation), a Merkle tree not binding the leaf count (fixed by binding n into the root), a settlement path that accepted a negative share and thus let one party push a counterparty below zero (fixed by outright rejection), and superficial invariants that verified nothing at all (replaced to read the real verdict, demonstrated by deliberately breaking the verdict and watching tens to hundreds of cases fail). **That verification passed means it could have failed and did not.**

## Honest limits

These eight slices are all **pre-testnet prototypes** and do not substitute for measurement. Specifically:

- **The slices are not yet one running system.** D1's DAG ledger, D2's beacon, and D3's wiring are each independent prototypes. D1 does not actually receive and process C's channel settlements, nor does D2's beacon actually drive B1's sampling. We showed that each abstraction holds **individually** as a real structure; an end-to-end system combining them all in one process is the province of the testnet.
- **Remaining abstractions**: adjudication is still a stub reflecting the measured FAR (real LLM re-execution is measured separately by B2), a VDF to remove the beacon's residual bias and P2P networking are roadmap, and A's economic model presumes four conservative simplifications.
- **Conditions for confirmation**: reproduction of the target detection rate and finalization time, the verification parameters, and the actual participation of external supply and verification nodes are confirmed by Phase 1 testnet measurement (whitepaper 10.3).
- We distinguished what the prototypes showed from what they did not, and no result is claimed beyond measurement.

## Reproducibility

Every slice has its code, tests, and results published in the repository, and reproduces deterministically (fixed seeds).

| Slice | Code | Results |
|-------|------|---------|
| A. Issuance economics | `sim/` (60 tests) | `results/` (sweeps, frontiers, regression) |
| B2. Re-execution adjudication | `verify/` (38 fast + @slow real-model) | `verify/results/` |
| Feedback | `sim/` FAR sweep | `results/far_feedback/` |
| C. Payment channel | `channel/` (35 tests) | `results/channel/` |
| B1. PoI plumbing | `poi/` (49 tests) | `results/poi/` |
| D1. DAG L1 ledger | `ledger/` (38 tests) | `results/ledger/` |
| D2. Randomness beacon | `beacon/` (24 tests) | `results/beacon/` |
| D3. PoI×adjudication wiring | `d3/` (21 tests, depends on poi for real) | `results/d3/` |

A total of **265 fast tests** plus real-model @slow experiments. Each slice includes its own README and findings report, and the design and implementation plans are in `docs/superpowers/{specs,plans}/`.

---

*This document is the validation record backing the whitepaper's claims. See `docs/whitepaper/` for the whitepaper body and each directory's README for the detail of each slice.*
