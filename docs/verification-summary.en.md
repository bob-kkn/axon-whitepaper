# Axon Design Validation Summary

> A record of validating the whitepaper's core claims through five independent prototypes. For investor and partner review.
> All results are pre-testnet prototype output; confirmation is left to testnet measurement.

## In brief

The Axon whitepaper makes three large claims — that the **issuance economics** of compute supply are safe against fraud, that the **verification** (re-execution adjudication) protecting that issuance is **actually feasible**, and that the **payment and verification plumbing** built on top **withstands attack**. Within the whitepaper these three axes are arguments, but we did not leave them as assertions. Five independent prototypes each realized a claim in code and stress-tested it under adversarial conditions, and all three axes passed a first-pass validation. What matters is that these prototypes hold each other up — one slice's measurement fills another slice's assumption, and yet another slice confirms that the assumption holds on the real plumbing. Below is an honest summary of what we verified, what held, and what still remains the province of the testnet.

## What we verified — the five slices

| Slice | Question verified | Method | Key result | Honest limit |
|-------|-------------------|--------|------------|--------------|
| **A. Issuance economics (`sim/`)** | Is the expected value of fraudulent issuance really negative at the draft parameters | Agent-based Monte Carlo + closed-form oracle regression | Under four attacks (adaptive, distributed, collusion, evasion) the ch7 defaults keep EV<0. **The center of gravity of the defense is detection, not the deposit** — cumulative detection converges to 1, so it holds even with the deposit driven extremely low | Four conservative model simplifications disclosed. The sole collapse point is full verifier collusion + disappearance of third-party re-checks |
| **B2. Re-execution adjudication (`verify/`)** | Is the re-execution adjudication that A *assumed* feasible actually feasible with real LLMs | Real-model experiment (Qwen2.5, SmolLM2, heterogeneous ollama) | Homogeneous runtimes adjudicate perfectly by regeneration (AUC 1.0), complementary with teacher-forcing. **Heterogeneous runtimes adjudicate imperfectly** — worst-case false-negative rate (FAR) ≈ 0.375 | A's "per-sample detection = 1" holds only for homogeneous runtimes; heterogeneity was an optimistic assumption, shown by measurement |
| **Feedback (`sim/` FAR)** | Does the economy hold once the imperfection of heterogeneous adjudication is injected | Feed B2's measured FAR (0.25–0.375) back into the A engine and re-sweep | The ch7 defaults keep EV<0 across **all 96/96 scenarios; no deposit increase needed.** Even as per-epoch adjudication leaks, the 24-epoch cumulative detection stays ≥0.997 | Adjudication imperfection modeled as a single parameter (correlated failure is a separate task) |
| **C. Payment channel (`channel/`)** | Is the honest party protected under exit fraud and offline exit in an L2 channel | Real ed25519 signatures + commit-close-dispute state machine | Latest-signed-state-wins dispute preserves the honest party's share under both attacks. **Four invariants hold over 500 arbitrary histories with 0 violations** | Seedless genesis, group channels, and real networking are out of scope |
| **B1. PoI plumbing (`poi/`)** | Does the commit-sample-slash plumbing of issuance verification withstand protocol attack | Real sha256 Merkle + public-randomness sampling + slashing | All four attacks (tamper, non-disclosure, inclusion forgery, grinding) resolve to slashing. Four invariants over 300+ epochs, 0 violations. **The measured detection rate reproduces A's oracle `1−(1−f)^k` within 3σ** | The actual ledger derivation of the public randomness and the re-execution adjudication are separate layers (replaced by abstract oracles) |

## How it all connects — the chain of validation

The five slices run independently, but they pass assumptions and measurements back and forth to form a single chain.

- **A assumed two things** — that the detection probability of sampling is `1−(1−f)^k` (the sampling oracle), and that re-execution adjudication is perfect.
- **B1 realized the first assumption.** When we built the plumbing with real Merkle commitments and post-commit public-randomness sampling, the measured detection rate over many epochs reproduced exactly the closed form A had premised. **A's economic conclusion stands not on an abstraction but on the real plumbing.**
- **B2 refuted the second assumption.** In real-LLM re-execution, heterogeneous-runtime adjudication was not perfect (FAR 0.25–0.375). So we injected that measured value back into A as **feedback** and re-swept — and even with imperfect adjudication, repetition and accumulation catch it downstream, so the economic defense still held.
- **C realized the payment path (Chapter 6), and B1 the issuance path (Chapter 7)**, showing by invariants that the dispute and slashing rules the whitepaper had deferred to "fixed at the implementation stage" actually protect the honest party.

In short, B2 measures reality and feeds A, B1 realizes A's sampling mechanism and confirms A, and C and B1 guard the plumbing of both paths. The five slices interlock.

## The three most important findings

1. **Security depends on detection, not on the size of the deposit.** The intuition is "grow the deposit and you are safe," but the simulation pointed to a different center of gravity. Issuance is a repeated game, so independent sampling repeats each epoch and the cumulative detection probability of a node that continues forging converges to 1. Thus the expected value stayed negative even with the deposit driven extremely low, and the only point where the defense actually collapsed was when verifiers all colluded and third-party re-checks vanished. **What bears the load of the design is not the collateral but the random assignment of verifiers and the openness of challenge.**

2. **Heterogeneous-runtime adjudication is imperfect, but the economy holds anyway.** Reproducing the same inference across different hardware and runtimes is hard because of floating-point nondeterminism, and a measured false-negative rate existed (worst-case ≈0.375). This refutes the optimistic assumption but is not a catastrophe — a single epoch's adjudication failure is caught in later epochs, the per-trial cumulative detection exceeds 0.997, and once caught in the end, deposit forfeiture overwhelms the gain. **Even with the measured condition injected, the economic defense of the issuance path holds.**

3. **Payment and plumbing security were verified by invariants and adversarial scenarios, and review caught real holes.** The security properties of the channel (C) and the PoI plumbing (B1) were not merely passed through a few scenarios but verified as **invariants** over hundreds to thousands of arbitrary histories and epochs. Each prototype went through an adversarial review before merge, and the review caught real defects — a cooperative close replaying a stale state (fixed by domain separation), a Merkle tree not binding the leaf count (fixed by binding n into the root), and a superficial invariant that did not actually verify the ledger (replaced to read the real verdict, demonstrated by deliberately breaking the verdict and watching tens to hundreds of histories/epochs fail). **That verification passed means it could have failed and did not.**

## Honest limits

These five slices are all **pre-testnet prototypes** and do not substitute for measurement. Specifically:

- **Abstractions**: the L1 ledger is an in-memory settlement engine (MockL1), the public-randomness seed is an abstraction of "unpredictable after commit," re-execution adjudication is an abstract oracle (B2 handles the measurement separately), and A's economic model presumes four conservative simplifications. The real DAG, consensus, VDF, and networking are outside this scope.
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

A total of **182 fast tests** plus real-model @slow experiments. Each slice includes its own README and findings report, and the design and implementation plans are in `docs/superpowers/{specs,plans}/`.

---

*This document is the validation record backing the whitepaper's claims. See `docs/whitepaper/` for the whitepaper body and each directory's README for the detail of each slice.*
