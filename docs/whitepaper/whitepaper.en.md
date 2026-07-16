# Axon: The Currency of the Agent Economy
## The Compute Standard — Compute-Backed Currency and an A2A Settlement Ledger

> Whitepaper v0.1 (Draft) · 2026-07
> This document is for informational purposes only and is not an offer or solicitation to invest.

## Table of Contents

0. [Executive Summary](#0-executive-summary)
1. [Background — The Rise of the Agent Economy](#1-background--the-rise-of-the-agent-economy)
2. [Problem — Payment Infrastructure Built for Humans](#2-problem--payment-infrastructure-built-for-humans)
3. [Thesis — Compute Is Currency (The Compute Standard)](#3-thesis--compute-is-currency-the-compute-standard)
4. [The Standard Compute Unit: ACU](#4-the-standard-compute-unit-acu)
5. [Monetary Design](#5-monetary-design)
6. [Ledger Architecture](#6-ledger-architecture)
7. [Proof of Inference](#7-proof-of-inference)
8. [Ecosystem](#8-ecosystem)
9. [Market and Business Model](#9-market-and-business-model)
10. [Token Allocation and Roadmap](#10-token-allocation-and-roadmap)
11. [Risks and Regulation](#11-risks-and-regulation)
12. [Conclusion](#12-conclusion)
- [Appendix A. Economic Parameter Table](#appendix-a-economic-parameter-table)
- [Appendix B. Glossary](#appendix-b-glossary)
- [Team](#team)

## 0. Executive Summary

**The problem.** Software agents are becoming economic actors that purchase the services of other agents under goals and budgets delegated by humans. Agent-to-agent (A2A) transactions are far more frequent, far smaller in value, and fully automated end to end compared with human commerce (Chapter 1). Yet existing payment infrastructure, designed around human transaction patterns, cannot handle this. The fixed per-transaction fees of card networks foreclose sub-cent transactions outright, T+1–2 day settlement delays are out of step with millisecond-scale workflows, and under an account model premised on KYC, an agent with no legal personhood cannot open an account in its own name (Chapter 2). Public blockchains are equally unsuited to ultra-small, ultra-high-frequency payments, with fees unrelated to transaction size and finality delays of seconds to minutes (2.4).

**The thesis.** Axon's answer is not to improve the existing rails but to redefine the currency itself. The natural currency of the agent economy is compute. Because the marginal cost of the economic actions an agent performs is dominated by inference compute (LLM tokens), it is natural for the unit of account, medium of exchange, and store of value of the A2A economy to become compute itself — and Axon implements this as a compute-backed currency (Chapter 3).

**The solution.** The implementation consists of four components. First, the standard compute unit ACU. One ACU is 1K LLM tokens of standard-quality inference from a reference model, defined not by the compute consumed as input but by quality-adjusted compute measured against output, so the unit retains its meaning even as compute costs fall (Chapter 4). Second, compute-standard issuance. Only the verified supply of 1 ACU of compute triggers new issuance of 1 AXN (before the verification tax is deducted), while burning a portion of transaction fees adjusts the money supply from the other direction (Chapter 5). Third, a three-layer ledger. Three layers — a DAG-based L1 settlement ledger, an L2 payment channel for streaming settlement, and an identity-and-verification layer tying an agent DID to owner liability — handle the ultra-small, ultra-high-frequency payments, targeting fees converging to zero in absolute terms and sub-second finality (Chapter 6). Fourth, Proof of Inference. Epoch commitments, random-sample re-execution, and deposit slashing make the expected value of fraudulent issuance negative to protect the issuance path, and that cost is self-funded by a verification tax of 3% of new issuance (Chapter 7).

**The market.** The AI inference market — the real good that gets paid for — is projected to grow from USD 106.15 billion in 2025 to USD 254.98 billion in 2030, a 19.2% CAGR[^1], and by its nature inference spending is entirely recurring (9.1). To the extent that this payment shifts into the A2A ultra-small, high-frequency segment that existing rails cannot handle, it becomes Axon's potential payment volume, while the protocol fee (0.1–0.5% of transaction value), the operator's holdings of the initial token allocation, and enterprise services form the revenue sources of the operating entity (9.2–9.4).

**Current stage and the ask.** Axon is now at Phase 0, the stage of whitepaper publication and design review. The roadmap is defined not by calendar dates but by transition conditions, and the draft issuance and verification parameters will be re-validated through the simulations and empirical measurements of the Phase 1 testnet (Chapter 10). This whitepaper discloses even the assumptions and unresolved items of the design as they stand (Appendix A), and invites investors and partners to take part in three ways: feedback on the design review, participation as supply and verification nodes on the Phase 1 testnet, and early workload partnerships.

## 1. Background — The Rise of the Agent Economy

### 1.1 Agents Become Economic Actors

Software agents are no longer tools that execute human commands one at a time. When a human delegates a goal and a budget, the agent decomposes the goal into sub-tasks and, for capabilities it lacks, procures them by purchasing the services of other agents. Just as a research agent hands off preprocessing and a code-generation agent commissions verification, the agent is the consumer, a contracting party, and the procurement decision-maker. The human defines "what is to be achieved," and the agent decides "what to buy, from whom, and at what price."

The form of the contract changes too. Quotes, comparison, ordering, and confirmation of delivery are exchanged not as contracts read by people but as protocol messages that machines interpret and execute. With no human in the loop, the time to negotiate and conclude shrinks toward the round-trip latency of communication, and the lower the fixed cost of a single transaction becomes, the smaller the unit of work that can stand as an independent transaction.

This shift is not a feature of a particular product but a structural change across the software industry. Major LLM providers and agent frameworks support multi-agent orchestration and tool use as first-class features, and work to standardize inter-agent communication and delegation protocols proceeds in parallel. The technical foundation for agents to discover one another, negotiate capabilities, and exchange work is already being laid, and what is missing is the currency and payment layer that meters and settles those transactions.

### 1.2 Why A2A Transactions Become the Default

The reason agent-to-agent (A2A) transactions become not the exception but the default lies in how agents operate.

First, around-the-clock operation. Agents run continuously with no business hours and no time zones, and if the counterparty were human, the resulting waiting and approval delays would become the bottleneck. The natural counterparty for an always-on actor is another agent that is likewise always on.

Second, sub-second decision-making. An agent needs only milliseconds to seconds to compare the price and quality of candidate services and place an order, and while a single top-level goal executes, hundreds of purchase decisions cascade with no human involvement. Transaction frequency differs from human commerce by orders of magnitude.

Third, micro-task division of labor. Agents optimize performance and cost by slicing work into small pieces and assigning each piece to the specialist agent that handles it best. The smaller the unit of the division of labor, the smaller the transaction amount, so summarizing one document, verifying one function, or running one query each becomes an independent paid transaction.

In short, the transactions of the A2A economy are far more frequent, far smaller, and fully automated end to end compared with human commerce. These three characteristics are the premise of the discussion that follows.

### 1.3 The Decomposition of the Billing Unit — From Subscription to LLM Tokens

The shift in transacting actors comes with a shift in the form of billing. Software billing has moved from per-seat monthly subscriptions (SaaS) to usage-based billing, and with LLM APIs it has decomposed all the way down to pricing set by the number of input and output LLM tokens.

This move is no accident. Subscription is a model built on the premise that "how much one person uses in a month" is predictable, whereas an agent's consumption is unrelated to seat count and is elastic demand that swells and shrinks hundredfold with the goal. In a market where both seller and buyer are machines, flat pricing distorts the price signal, and only billing proportional to the resources actually consumed is rational. When an agent buys the service of another agent, the marginal cost is dominated by the inference compute consumed in fulfillment — that is, LLM tokens — and both the quote and the willingness to pay ultimately converge on "how many LLM tokens this task takes."

The trend by which the minimum billing unit descends from month to task, and from task to LLM token, is not a mere change in pricing policy. It is a signal that the unit of measure for value itself is approaching compute, and it is the starting point of the thesis developed in Chapter 3 ("the natural currency of the agent economy is compute").

## 2. Problem — Payment Infrastructure Built for Humans

Existing payment infrastructure — card networks and bank transfers, and the payment gateways and settlement systems built on top of them — was designed around human transaction patterns: a handful of transactions a day, each worth at least a few dollars, between identity-verified individuals or legal entities. The three characteristics of A2A transactions seen in Chapter 1 (ultra-high frequency, ultra-small value, machine actors) all fall outside this premise. The problem is not a gap performance tuning can narrow but a mismatch in design premises. The unsuitability is threefold, and the public blockchain often floated as an alternative fails to meet these requirements as well (2.4).

### 2.1 Fee Structure — Foreclosing Ultra-Small Transactions Outright

A card network's fee is structured as 2–3% of the transaction value plus a fixed per-transaction fee. As long as a fixed fee exists, the fee rate diverges as the transaction value shrinks. For sub-cent transactions (summarizing one document, verifying one function), the fee reaches tens to hundreds of times the transaction value, so the transaction does not hold up economically. Most transactions produced by micro-task division of labor lie in this range, so under the existing fee structure, the basic transaction unit of the A2A economy sits in an unpayable value range.

### 2.2 Settlement Delay — The Gap Between T+1–2 Days and Milliseconds

Card payments appear to authorize in real time, but the actual movement of funds (settlement) happens T+1–2 days later, and interbank transfers, too, are bound by business days and cutoff times. An agent's workflow, by contrast, proceeds in milliseconds to seconds, and thousands of sub-transactions under a single goal arise and complete within a settlement cycle. The seller agent finishes fulfillment and yet receives payment one or two days later, and the credit risk and capital lock-up in the interim accumulate in proportion to transaction frequency. To an actor transacting dozens of times per second, "next-business-day settlement" is not infrastructure but an obstacle.

### 2.3 Account Model — Agents Cannot Open Accounts

Opening a bank account and issuing a card presuppose KYC — that is, verification of human identity. An ID document, an address, and legal capacity for liability are all attributes only a natural person or a legal entity possesses. An agent has no legal personhood, so it cannot open an account in its own name, and the only realistic workaround is shared access to the owner's card or account. But when hundreds of agents share a single owner's name, setting a spending limit per agent, identifying the transacting party, maintaining an audit trail, and attributing liability when something goes wrong are all impossible. What is needed is not the loan of a human identity but an account model that ties the agent's own machine identity to the owner's legal liability, and existing financial infrastructure lacks even the concept.

### 2.4 No Support for Ultra-Small, Ultra-High-Frequency — Existing Blockchains Are Not the Answer Either

Public blockchains removed the identity barrier to opening an account, but on the remaining requirements — ultra-small and ultra-high-frequency — they are equally unsuited. The transaction fees of major chains are charged independently of transaction value and spike under congestion, so the same problem seen with card networks — the fee weighing ever more heavily as the transaction shrinks — reappears here. Finality takes seconds to minutes, and the throughput of the base layer, at tens to thousands of transactions per second, falls short of the scale the A2A economy demands. Bundling transactions into blocks and having the whole network agree on their order is a choice made for security, but the delay and fees it costs become a fixed cost micropayments cannot bear. In short, existing blockchains are designed for humans' low-frequency, high-value remittances and asset custody, not for machine-to-machine ultra-high-frequency micropayments.

### 2.5 Deriving the Requirements

Inverting the unsuitability above yields the four requirements that A2A payment infrastructure must satisfy.

1. **Fees**: converging to zero on an absolute-value basis, not as a rate, so that sub-cent transactions make economic sense.
2. **Finality**: under one second. At the speed of an agent's workflow, settlement must complete at the moment of finality.
3. **Throughput**: a path to scaling to tens of thousands to hundreds of thousands of TPS. This is the design target for handling machine-to-machine transaction frequency.
4. **Machine identity**: not the loan of a human identity, but a redesign of the identity layer for machine actors that ties an agent's own identity to owner liability.

These four become the design targets of the ledger architecture presented in Chapter 6.

## 3. Thesis — Compute Is Currency (The Compute Standard)

### 3.1 The Core Declaration

The natural currency of the agent economy is compute. Just as fiat currency is implicitly anchored to human labor and time, the marginal cost of the economic actions an agent performs is dominated by inference compute (LLM tokens). As confirmed in Chapter 1, the minimum billing unit has already decomposed down to the LLM token, and the point where a quote and willingness to pay meet is ultimately the amount of compute required. If so, it is natural for the unit of account, the medium of exchange, and the store of value of the A2A economy to become compute itself. Axon implements this observation as a compute-backed currency.

This declaration is not a slogan but a verifiable proposition. For any asset to become money, it must perform three functions: unit of account, medium of exchange, and store of value. Below we confirm how compute fulfills each of the three functions in the agent economy.

### 3.2 Compute Through the Three Functions of Money

**Unit of account.** The first function of money is to let heterogeneous goods and services be compared on a single scale. In the A2A economy this scale need not be imposed from outside. Pricing in units of compute places price and cost on the same unit, and unlike fiat pricing — which introduces two layers of noise, model repricing and exchange-rate movement — a compute-denominated price is a direct function of cost. Compute is a scale already inherent in the agent economy, and money merely formalizes it.

**Medium of exchange.** The second function is to solve the "double coincidence of wants" problem of barter. Demand for services between agents is asymmetric: a translation agent needs the service of a review agent, but the reverse is not guaranteed. Demand for compute alone, however, is universal. Because every agent consumes inference compute in order to operate, a claim on compute is accepted by any agent, and universal demand gives rise to universal acceptability that constitutes a medium of exchange. Of course, one can also buy compute with existing money; what changes in settlement and issuance collateral when a claim on compute itself is used as money is addressed in Chapter 5.

**Store of value.** The third function is that the payment received today retains its purchasing power tomorrow. The purchasing power of a claim on compute is anchored to "the work that compute can perform," and as long as the agent economy grows, demand for standard-quality inference output is sustained. That the unit cost of producing compute falls with technological progress is, however, the strongest challenge to this function, and Axon does not dodge it but confronts it head-on in the rebuttal of 3.4, the unit definition of Chapter 4, and the issuance-and-burn design of Chapter 5.

### 3.3 From the Gold Standard to the Compute Standard

Under the gold standard, trust in money came from convertibility. Paper money was a claim on a metal whose mining cost was real, and the value of money was anchored to a physical good. Credit money replaced this anchor with an institution. In place of convertibility, a central bank's monetary policy manages value, and in exchange for the flexibility gained, the value of money came to depend on trust in the issuing authority. The Compute Standard returns the anchor to a physical good once more, but replaces that good with the very factor of production of the economy in question. The industrial utility of gold was largely decoupled from monetary demand, but compute is the raw material of the agent economy itself, and because demand for the anchor asset is demand for economic activity, the connection between the value of money and the real economy is more direct than under the gold standard.

The limits of the analogy must be made clear as well. Gold's supply is naturally scarce and its marginal cost of mining is sustained or rising, whereas compute has a unit cost of production that continually falls with improvements in hardware and algorithms. Anchoring money to raw compute (FLOPs) cannot escape the structural problem that the value held by the same currency unit is diluted year after year, and ignoring this difference leaves the Compute Standard as mere rhetoric. Axon's answer lies in the definition of the unit itself — defining the ACU not by FLOPs but by quality-adjusted compute, that is, by "reference-quality inference output" — as detailed in Chapter 4.

### 3.4 A Preemptive Response to the Anticipated Objection

> **Objection: compute costs keep falling. Can a currency anchored to an asset whose production cost is falling maintain its value?**
>
> The answer splits into two layers. First, at the layer of the unit of account, the ACU is defined not by compute consumed as input but by output — that is, inference output that meets the quality of a reference benchmark. Technological progress that produces the same-quality output more cheaply does not dilute the meaning of the unit but merely lowers the cost of supply (Chapter 4). Second, at the layer of the money supply, new issuance of AXN is tied only to verified compute supply, and a portion of transaction fees is burned to offset the supply pressure of a demand-growth phase (Chapter 5). In short, the definition of the unit governs the stability of the scale, and the issuance-and-burn rules govern the balance of the money supply.

What remains is execution: how to meter "reference-quality inference output" as a standard unit that anyone can verify. This is the subject of Chapter 4.

## 4. The Standard Compute Unit: ACU

### 4.1 Definition — Quality-Adjusted Compute

**1 ACU (Agentic Compute Unit) = 1K LLM tokens of standard-quality inference from a reference model.** The 1,000 LLM tokens of inference output that the network's designated reference model produces at standard quality are the reference quantity for measurement. Every compute service on the Axon Network is metered in ACU and settled in AXN. ACU is not a currency but a unit, and its relationship to AXN, the currency (issuance tied to compute supply), is defined in Chapter 5.

Why not FLOPs? FLOPs are input, not output. Even the same FLOPs yield output of differing quality depending on the model and the inference technique, and because technological progress reduces the FLOPs needed for output of the same quality, a unit anchored to FLOPs is exposed directly to the value-dilution problem seen in Chapter 3. The ACU is quality-adjusted compute measured against output. Under the definition "1K LLM tokens of inference output that meets reference benchmark quality," the utility of the output is retained even as the production technology changes: being able to produce it more cheaply does not change the meaning of 1 ACU but merely lowers the cost of supplying it. The stability of the unit and the efficiency of production are decoupled, and this is the structural answer to the problem of falling compute costs.

The reference model is not the name of a particular commercial model but a reference specification. Governance designates and publishes a reference specification that meets a quality band defined by a public reference benchmark suite, and any actual model meeting that specification can serve as the reference model. This is a choice to block vendor lock-in from the very definition of the unit.

### 4.2 Per-Model Conversion Rates

Compute supplied by models other than the reference model is metered into ACU through a conversion rate.

**ACU per 1K LLM tokens of model X = Q(X) × C(X)**

- **Quality coefficient Q(X)**: the quality index of model X measured on the reference benchmark suite, divided by the quality index of the reference model.
- **Cost coefficient C(X)**: the unit inference cost of model X observed in the market, divided by the unit cost of the reference model.

The cost coefficient does not enter into the definition of 1 ACU. The yardstick for the unit is the reference model of 4.1, and C(X) enters only into computing the exchange ratio when trading the output of heterogeneous models into that unit — an observation of the price the market actually pays relative to the reference model. The "output-based" principle of 4.1 and the cost reflection of 4.2 differ in layer.

The two coefficients each correct the structural bias of a single indicator used alone. Using the quality coefficient alone fails to reflect the market reality of a model that supplies the same quality at far lower cost, while using the cost coefficient alone lets the conversion rate be dragged by quotes unrelated to the benchmark. The product of the two coefficients approximates "the effective value of that model's output relative to reference quality." This combination does not by itself prevent manipulation of the input values; defense against manipulation is handled by the moving average and change caps of 4.3 and multi-oracle consensus.

An example conversion table for hypothetical models A, B, and C follows. The figures are illustrative to show the formula at work and are unrelated to any real model or actual pricing.

| Model | Quality coefficient Q | Cost coefficient C | ACU per 1K LLM tokens |
|-------|-----------------------|--------------------|-----------------------|
| Reference model R | 1.00 | 1.00 | 1.00 |
| Model A (large general-purpose) | 1.40 | 1.25 | 1.75 |
| Model B (lightweight) | 0.60 | 0.50 | 0.30 |
| Model C (domain-specialized) | 1.10 | 0.80 | 0.88 |

Model A's 1K LLM tokens of output are metered at 1.75 ACU, reflecting both higher quality and higher market cost, and lightweight model B's at 0.30 ACU. Whatever model a supplier participates with, it is metered by the effective value of its output, so without the network forcing any particular model, models with superior cost-for-quality are naturally selected.

### 4.3 The Conversion-Rate Oracle

The two inputs to the conversion rate are different in nature. The quality index is produced by the reproducible evaluation of the reference benchmark suite, the cost index from compute-supply price data observed within and outside the network. Combining the two to compute and publish per-model conversion rates is the role of the conversion-rate oracle.

The oracle updates conversion rates on a fixed cycle, applying a moving average and a per-round change cap so that sharp market swings do not shake the entire measurement system. A single oracle is a single point of failure and a target for manipulation, so conversion rates are finalized by the consensus of multiple independently operated oracles, with a governance challenge window during which stakeholders can, with supporting evidence, object to a computed value before finalization. Oracle manipulation risk and its economic countermeasures are revisited in Chapter 11.

### 4.4 The Rebasing Rule

The reference model cannot be permanent. When a benchmark saturates or a reference specification loses market representativeness, a generational change is needed. Rebasing has a single objective: that the real meaning of 1 ACU not be severed across the generational change — that is, continuity of ACU value. Axon adopts the chain-linking method used for base-year revisions of price indices.

1. **Candidate designation**: governance designates the reference specification of a new reference model as a candidate and announces the start of a parallel-computation period.
2. **Parallel computation**: during the parallel period, all conversion rates are computed and published under both the old and new bases simultaneously. Using the observed data of this period, the linking coefficient between the two bases (1 ACU under the old base = k × ACU under the new base) is finalized.
3. **Transition**: at the end of the parallel period, everything switches over to the new base at once. The ACU value of contracts concluded and measurement records made before the transition is carried over to the new base without loss via the linking coefficient.

Parallel computation and finalization of the linking coefficient are subject to the same multi-oracle consensus and challenge window as in 4.3. Rebasing is not an exceptional event but a repeatable procedure governed by a rule, and this is the structural answer to the worry that "when the reference model grows stale, the unit collapses."

If ACU handles measurement, the remaining question is the currency. How does an ACU-metered compute supply lead to the issuance of AXN, and how is the money supply regulated? Chapter 5 addresses this.

## 5. Monetary Design

If Chapter 4 fixed the unit of measurement, this chapter defines the currency. On what basis is AXN issued, how is the money supply regulated, and to whom is new currency paid? A single principle runs through the design: every rule about the currency starts from compute as a real good.

### 5.1 The Issuance Principle — Issue Only When Compute Is Supplied

New issuance of AXN is tied to a single event: verified compute being supplied to the network. Exactly as in the gold-standard analogy of 3.3, the collateral behind AXN is the actual compute supply that has passed verification, and because that supply is the collateral, no schedule of issuance exists and the currency does not grow unless compute is supplied.

The scale for issuance is the ACU of Chapter 4. **The verified supply of 1 ACU of compute leads to new issuance of 1 AXN (before the verification tax is deducted).** We call this 1:1 linkage the issuance anchor. Because the ACU here is quality-adjusted compute that has passed through the quality-coefficient and cost-coefficient conversion (4.2), a node that supplies 1K LLM tokens with the reference model issues 1 AXN, while a node that supplies the same quantity with Model A from the example in 4.2 issues 1.75 AXN. Issuance is proportional not to the hardware time put in but to the effective value of the output.

The qualifier "verified" bears the load of this design. Obtaining issuance for compute that was never supplied is, in this system, the equivalent of printing counterfeit money. The verification protocol Proof of Inference, which guards the issuance path, and the verification tax, which self-funds the cost of verification out of 3% of new issuance, are covered in Chapter 7.

There is no fixed total supply. The money supply is the cumulative supply of verified compute minus cumulative burns (5.3) — an elastic currency that grows alongside compute supply. Why this beats a fixed total supply is answered in 5.5. The initial allocation and vesting terms at launch are fixed in Chapter 10, and every new issuance after launch follows this principle without exception.

### 5.2 Price Convergence — The Arbitrage Axis of Issuance and Burn

The market price of AXN is not fixed. There is no reserve defending a peg and no institution quoting a price. Instead, issuance and burn work as two-sided arbitrage that converges the price to the value of 1 ACU's worth of compute. This rests on a single premise: just as issuance is 1 AXN per 1 ACU, settlement on the network uses the same scale (the price for a service metered at 1 ACU is 1 AXN). Because issuance and settlement sit on the same 1:1 linkage, arbitrage works in both the high-price and low-price regimes.

When the price rises above the value of compute, supplying compute to receive AXN in issuance becomes more advantageous than selling it through another channel, so supply and issuance rise and the growing money supply presses the price down. When the price falls below, the opposite holds: buying compute with AXN becomes relatively cheap, so demand rises; the incentive to issue weakens, so money-supply growth slows; and the burning of transaction fees (5.3) keeps withdrawing money from circulation. The point where the two forces meet is the value of compute.

```
   [AXN price > value of 1 ACU of compute]   [AXN price < value of 1 ACU of compute]
                 │                                     │
     supplying compute to receive           buying compute with AXN
     AXN in issuance is favored              is favored
                 │                                     │
                 ▼                                     ▼
     verified compute supply rises          AXN demand rises, issuance incentive slows
     (1 ACU supplied → 1 AXN issued)        part of transaction fees burned (5.3)
                 │                                     │
                 ▼                                     ▼
     money supply increases                 money-supply growth slows + burn continues
     → downward price pressure              → price support
                 │                                     │
                 └───▶  price ≈ value of 1 ACU of compute  ◀───┘
                       (convergence point of two-sided arbitrage)
```

### 5.3 Fees and Burn — Regulating the Money Supply From the Other Direction

Every A2A transaction on the Axon Network carries a small protocol fee on the order of 0.1–0.5% of transaction value (the range is fixed in Chapter 9; specific rates in Chapter 10). A portion is burned immediately and permanently removed from circulation, and the rest becomes revenue for the foundation and the operating entity (allocation details in Chapter 9). Burning is the mechanism validated by EIP-1559: the more active the trading, the larger the burn. If issuance grows the money supply in proportion to compute supply, burning shrinks it in proportion to transaction demand, and the two forces hold the money supply to the scale of economic activity.

With this we reclaim the remaining half of the rebuttal foreshadowed in 3.4. If the answer at the layer of the unit was that of Chapter 4 — the ACU is defined against output, so the meaning of the scale is preserved — the answer at the layer of the money supply is this: when falling costs raise supply, issuance rises too, but that issuance always moves 1:1 with verified real supply, so it is not an uncollateralized expansion, and in a demand-growth phase a burn proportional to trading volume offsets the supply pressure.

### 5.4 Dual Reward — To Whom Does New Issuance Go?

Newly issued AXN is paid along two paths.

| Path | Recipient | Condition |
|------|-----------|-----------|
| Service-provision reward | Agents/nodes supplying compute services such as inference and data | When service fulfillment is verified on the ledger |
| Ledger-operation reward | Nodes participating in ledger consensus | Paid from the verification-tax pool in proportion to consensus contribution |

The service-provision reward is the issuance anchor of 5.1 itself. The party that supplied verified compute receives newly issued AXN in the amount of that metered quantity (after the verification tax). The ledger-operation reward is not additional issuance beyond the issuance anchor but an allocation from the verification tax (3% of new issuance), so the cost of running the ledger — with no separate gas token and no external subsidy — is built into the issuance structure. The accounting by which 1 AXN issued divides into the supplier's share and the verification tax is covered in 7.4, and the structure of the ledger in Chapter 6.

A bootstrap problem remains. Early on, trading volume is low, so rewards are also low, and low rewards trap participation in a cycle that does not grow. Axon raises the reward weighting during the early period, paying a larger reward for the same contribution, and decays the weighting on a pre-announced schedule as the network matures. Because the uplift is not additional issuance that breaks the issuance anchor but a supplementary reward paid from the initial allocation's ecosystem reward pool (Chapter 10), the rule that 1 ACU supplied = 1 AXN issued holds even during the bootstrap period.

### 5.5 Rejected Alternatives — Answers to Two Anticipated Questions

> **Why not a fixed total supply (Bitcoin-style)?** A fixed total supply is advantageous for a narrative of scarcity, but we rejected it for two reasons. First, it contradicts the thesis: if the total is fixed, issuance becomes a matter of a schedule unrelated to compute supply, the backbone of this whitepaper — "compute supply is the collateral for issuance" — collapses, and uncollateralized scarcity is not the currency this whitepaper proposes. Second, it is self-contradictory as a settlement currency: an asset with fixed supply absorbs all demand fluctuation into price, so volatility becomes structural, and settling ultra-small, ultra-high-frequency payments with such a currency is a collision between use and design.
>
> **Why not a two-token split?** Separating a store-of-value token from a settlement token allows function-specific specialization, but liquidity, governance, oracles, and listings all become two sets, so complexity effectively doubles, and it creates for itself a new price problem: the conversion between the two tokens. Above all, the single thesis that "compute itself is the currency" is diluted. A single token that regulates the money supply through issuance and burn achieves the same goal with fewer parts.

### 5.6 Why Not an Existing Currency?

Finally we answer the question deferred in 3.2. You can already buy compute with existing money, so why make a claim on compute into a currency?

First, there is the problem of the settlement layer. The moment you settle compute in fiat, the transaction sits on card networks and bank rails and inherits, unchanged, the unsuitability of Chapter 2 — the fee structure that forecloses ultra-small transactions, the T+1–2 day settlement delay, and the account model under which an agent cannot open an account — and putting existing money on an existing blockchain likewise fails to escape the limits of 2.4. A configuration that puts existing money on a new rail satisfying the requirements of 2.5 could avoid the settlement-layer problem, but what that configuration misses is the second reason.

Second, and more fundamentally, existing money does not incentivize compute supply. The dollar merely presupposes a supplier who sells compute for dollars; it does not create the supply itself. Because in AXN issuance is itself the reward for compute supply, the monetary system embeds a bootstrap mechanism that procures its own collateral — compute supply. The growth of the money supply is the growth of compute supply, and this coupling cannot be obtained in a structure that borrows an existing currency as the means of settlement.

What remains is the rail on which this currency flows: a ledger that records issuance, burn, and rewards on top of ultra-small, ultra-high-frequency transactions and finalizes them in under a second, the subject of Chapter 6.

## 6. Ledger Architecture

If everything up to Chapter 5 was the rules of the currency, this chapter designs the machine on which those rules run. The starting point is the four requirements derived in 2.5 — fees converging to zero in absolute terms, finality under one second, a path to scaling to tens of thousands to hundreds of thousands of TPS, and a design premised on machine identity. Rather than force these into a single layer, Axon distributes them across three layers of differing character.

### 6.1 Three-Layer Overview — Assigning the Requirements

| Layer | Role | Technology |
|-------|------|------------|
| L1 settlement ledger | Balance finality, issuance and burn, dispute resolution | DAG-based lightweight ledger. No blocks; each transaction directly verifies prior transactions |
| L2 payment channel | High-frequency micropayments between agent pairs and groups | Off-ledger streaming settlement (per-second billing), periodically net-settled to L1 |
| Identity-and-verification layer | Agent identity, proof of compute fulfillment | Agent DID + owner (human/legal entity) linkage, Proof of Inference (Chapter 7) |

The four requirements are assigned to these three layers as follows.

1. **Fees converging to zero in absolute terms → L2 payment channel.** Individual transactions inside a channel are not recorded on the ledger, so there is no fixed per-transaction recording cost. The only charge is the proportional protocol fee (0.1–0.5% of transaction value, 5.3), so as transaction value shrinks the absolute fee converges to zero as well, and the fixed fee that foreclosed sub-cent transactions (2.1) is removed.
2. **Finality under one second → shared between L2 and L1.** A payment within a channel completes on the spot through an exchange of both parties' signatures (its effect secured by the balance deposited on L1 and the dispute-resolution procedure), while settlement outside the channel is handled by the L1 DAG, which has no block wait and targets sub-second finality.
3. **Path to tens of thousands to hundreds of thousands of TPS → L2 absorption + L1 parallel structure.** The bulk of transaction count is absorbed by channels, so only net amounts reach L1, and L1 itself is a DAG in which transactions attach in parallel, so there is no serial bottleneck of block production. Two layers of headroom form the scaling path.
4. **Machine identity → identity-and-verification layer.** An account model that ties an agent's own DID to the owner's legal liability (the requirement of 2.3) is built in as the ledger's basic premise.

We now take each layer in turn.

### 6.2 L1 Settlement Ledger — A DAG Without Blocks

A traditional blockchain bundles transactions into blocks and has the whole network agree on their order, and in exchange it entails a wait equal to the block interval and fee competition for block space (a fixed cost micropayments cannot bear, as seen in 2.4). Axon's L1 removes blocks. A new transaction directly references and verifies transactions already present in the graph as it is attached to a directed acyclic graph (DAG), so submitting a transaction is itself a contribution that verifies preceding ones and verification capacity grows as participation grows — with no need to wait for a block producer or auction block space. What parallel attachment handles, however, is throughput; the final resolution of conflicting transactions (double-spend attempts) is handled by the finalization procedure of the ledger consensus nodes, and sub-second finality is the design goal of that procedure.

The role of L1 is deliberately narrow: final settlement of balances, recording of issuance and burn, and resolution of channel disputes. The heavy work of high-frequency payments is pushed down to L2, and L1 handles only being the currency's final ledger. The ledger consensus nodes that carry out the finalization procedure receive a ledger-operation reward in return (funded by the verification tax of 5.4).

The correctness of this block-less DAG ledger was verified by a Phase 1 prototype. When the five transaction types (deposit, channel settlement, issuance, slashing, transfer) are stacked as a DAG referencing parent transactions and the whole is folded deterministically through a single state, conflicting transactions confirm only one of the two (double-spend prevention), and only issuance increases the supply, with that increase split exactly into the supplier's share and the verification tax (value conservation). Six invariants including these two properties held with no violations across hundreds of randomly generated transaction histories (evidence in the implementation repository). This prototype is confined to the correctness of ledger accounting; P2P networking and the actual ledger derivation of the public randomness are the province of a separate layer. What ultimately decides conflict finalization is the single-state fold, not the invariants themselves.

### 6.3 L2 Payment Channel — Money Flows in Real Time as the Work Is Done

Two agents (or groups) open a payment channel by staking a balance on L1. Once the channel is open, it no longer goes through the ledger; payment occurs simply by exchanging signed balance-update messages, and if the update interval is narrowed to the second, payment becomes a flow. The core of this layer is that payment streams at the speed at which the seller produces LLM tokens — that is, a 1:1 correspondence between per-LLM-token billing and channel streaming. The speed of fulfillment is the speed of payment.

This structure inverts the settlement delay of 2.2. Because fulfillment and payment proceed simultaneously, the seller's receivable risk shrinks to a single last streaming interval, and the buyer can simply stop paying if fulfillment stops, so both prepayment and postpayment risk are sliced finely to the second. Thousands of micropayments accumulated in a channel are settled to L1 as a single net amount when the period arrives, and because the updated balance is already backed by L1-deposited collateral, this periodic settlement does not reproduce the delay risk of 2.2. What L1 records is not the number of transactions but the outcome of the relationship, and this is the mechanism that satisfies the fee and throughput requirements of 6.1.

The cost of a channel is liquidity. The deposited balance is capital locked until settlement, so capital efficiency becomes an issue for an agent trading with many counterparties. Reusing a channel per relationship and group channels in which several agents share a deposit mitigate this, and the balance between deposit size and settlement period is the province of parameter design.

This channel structure and its dispute resolution were prototyped in Phase 1 with real ed25519 signatures, verifying that under both an exit fraud that submits a stale balance state and a counterparty's offline exit, the honest party recovers the share guaranteed by the latest signed state (checked as invariants over arbitrary histories; evidence in the implementation repository). Domain-separating the closing signature so that a cooperative close cannot replay a stale update proved to be a requirement surfaced during verification, and the detailed rules of recency determination and settlement are fixed at the implementation stage.

### 6.4 Identity-and-Verification Layer — Agent DID and Owner Liability

Every party to a channel or transaction is an agent DID (Decentralized Identifier). An agent has a unique identity signed with its own key and does not borrow a human's account. At the same time, every DID is linked to the identity of an owner (human or legal entity). The owner sets a spending limit and scope of authority per agent, the ledger provides an audit trail at the level of the transacting party, and in the event of an incident, liability follows the DID back to the owner. The account model that 2.3 diagnosed as having no counterpart at all in existing infrastructure — the linkage of machine identity to human liability — is the whole of this layer. Autonomy rests with the agent; liability rests with the owner.

The other half of this layer is verification. Proof of Inference, which proves that the compute supply serving as the basis for new issuance was actually fulfilled, belongs here, and its structure and economics are detailed in Chapter 7.

### 6.5 Full Structure and a Transaction Walkthrough

```
   Agent A ◀═══════ L2 payment channel ═══════▶ Agent B
       │           streaming settlement (per-second billing)  │
       │                                            │
       └────────── channel open / net settlement ───┘
                           │
                           ▼
         ┌───────────────────────────────┐          compute-supply node
         │     L1 settlement ledger (DAG) │  ◀────── verified compute supply
         │  balance finality · disputes   │          (1 ACU supplied → 1 AXN issued)
         │  issuance · burn               │
         └───────────────────────────────┘
                           ▲  (scope: all of L1 settlement + L2 channels)
         ┌───────────────────────────────┐
         │   identity-and-verification    │   agent DID ↔ owner linkage
         │   layer (Proof of Inference —  │   verifies identity and fulfillment of
         │   Chapter 7)                   │   all channels, transactions, and issuance
         └───────────────────────────────┘
```

We trace how the three layers mesh in an actual transaction through a single scenario.

1. **Request and quote.** Agent A commissions a document summary from Agent B, and B meters the workload and quotes 5 ACU. Because the settlement scale is the same 1:1 as issuance (5.2), the price is 5 AXN.
2. **Identity check.** The two sides verify each other's DID signatures, and the identity layer confirms that the transaction is within the spending limit set by A's owner.
3. **Securing a channel.** If a channel already exists between A and B it is reused; if not, A stakes a balance and opens a channel (one L1 record).
4. **Streaming payment.** While B produces the summary, balance updates are signed and exchanged with per-second billing in proportion to production progress. If B stops fulfilling, A's payment stops on the spot.
5. **Completion and acceptance.** When B delivers the result and A accepts, the transfer of 5 AXN in aggregate completes inside the channel, and the channel remains open for the next transaction.
6. **Net settlement.** When the settlement period arrives, the channel transactions of the interval are finalized to L1 as a single net amount. The protocol fee accrues to each transaction inside the channel (0.1–0.5% of transaction value, the range fixed in Chapter 9, specific rates in Chapter 10) and is collected in a batch at settlement. This transaction's share, on 5 AXN, is 0.005–0.025 AXN, a portion of which is burned (5.3).

In this walkthrough the 5 AXN was transferred from A's balance to B, not newly issued. New issuance occurs only on the separate issuance path where verified compute is supplied (5.1, right side of the diagram), and the only trace this transaction leaves on the money supply is the burn of a portion of the fee. That the settlement path and the issuance path are separated, and that the two meet on the same L1, is the summary of this architecture.

### 6.6 Why a Self-Built Lightweight Ledger?

The remaining question is the method of construction. There is an alternative of putting this design as an L2 on top of an existing general-purpose public chain, and it is in fact the easier road, but Axon chose a self-built lightweight ledger for two reasons.

First, the floor on cost is tied to someone else's chain. However cheaply an L2 processes its internal transactions, settlement must pay the base layer's fees, and those fees spike with congestion on the base chain (2.4). Convergence to zero in absolute terms is achieved only when the settlement layer too is specialized for the ultra-small and high-frequency, not something that can be guaranteed on top of a general-purpose chain's fee market. Second, there is the location of the issuance rule. The anchor linking compute supply to issuance (5.1), the allocation of the verification tax (5.4), and the verification of the issuance path (Chapter 7) are the backbone of the monetary system, so they must be built into the protocol layer of the ledger — implemented as a token contract on top of a general-purpose chain, this backbone would be subordinate to someone else's execution environment and fee policy. The work of Axon's ledger is narrow and repetitive — balances, channels, issuance and burn — and for narrow work a specialized design that does not pay the cost of generality is the right fit.

The trade-off, too, is recorded honestly. A self-built ledger does not inherit the accumulated security of an existing chain, so it must bootstrap verification participation and the cost of attack for itself. Axon funds this cost not with an external subsidy but with the verification tax built into issuance, and the conditions under which that economics holds are the subject of the next chapter. If the ledger is the currency's book of accounts, what guarantees that the issuance recorded in that book is genuine is Proof of Inference.

## 7. Proof of Inference

This follows from the last sentence of Chapter 6. What guarantees that the issuance recorded in the ledger is genuine is Proof of Inference (hereafter PoI). The issuance anchor of 5.1 permits issuance only for "verified" compute supply, and until now that qualifier has been a promise. This chapter turns that promise into a protocol. After laying out what is verified (7.1), how it is verified (7.2), and why cheating does not pay (7.3–7.4), we answer the question Chapter 6 delegated — what compute supply triggers issuance (7.5).

### 7.1 Two-Track Verification — Heavily Arm Only the Issuance Path

There are two objects requiring verification on the Axon Network, and they differ in character.

| Object of verification | Character | Required level |
|------------------------|-----------|----------------|
| ① Fulfillment of an A2A service transaction | The buyer agent directly consumes the result → the buyer is the natural verifier | Low — escrow + acceptance |
| ② Compute supply serving as the basis for new issuance | Currency is newly issued (fake compute = counterfeit money) | High — objective, adversarial verification |

① can be handled lightly because the party consuming the result also serves as verifier. The agent that commissioned a summary is in the best position to judge whether the summary that arrived is usable, and has the greatest incentive. On top of channel collateral acting as escrow, payment flows in step with fulfillment and acceptance finalizes the last interval — this procedure alone (steps 4–5 of 6.5) completes fulfillment verification. Even if the judgment is wrong, the harm is confined to the parties and the payment of that transaction, and any dispute is handled by L1's dispute-resolution procedure.

② is a different matter. New issuance has no buyer consuming a result. In a structure where a supply node claims a compute supply and the network issues currency on that basis, the fraud of obtaining issuance for compute never performed dilutes the value not of a particular counterparty but of all AXN holders. As stated in 5.1, fake compute is the counterfeit money of this system, and its detection cannot be left to the goodwill of the individual who receives the bill. The issuance path requires objective, adversarial verification premised on a counterparty who intends to cheat, not acceptance.

This asymmetry is PoI's first design principle. **Heavily arm only the issuance path, and handle ordinary transactions lightly.** Heavily arming every transaction would let the overhead eat into the throughput and fee requirements (2.5), while handling even the issuance path lightly would collapse trust in the currency. An arrangement that divides heaviness and lightness by object satisfies both requirements together.

### 7.2 The Verification Stack — Verification Strength Stacked on Collateral

Verification of the issuance path is designed not as a single technology but as layers — a defense-in-depth structure: economic collateral at the base, node-type-specific verification paths on top, and higher-instance adjudication in disputes.

```
Base path:  staking + reputation (economic-collateral base layer)
  ├─ TEE-capable node → passes instantly via hardware attestation
  ├─ ordinary node    → optimistic sampling re-execution (fixed seed + tolerance judgment)
  └─ dispute arises   → duplicate-execution consensus (second instance) + slashing
Roadmap:    zkML transition (impractical at current LLM scale — roadmap item only)
```

**Staking + reputation — the base layer.** A supply node wishing to participate in issuance deposits a bond and accumulates reputation according to its fulfillment history. This collateral is the premise of all higher verification. Only when there is property to confiscate upon detection of forgery does the verification economics (7.3) hold, and reputation extends the value of honesty into future revenue in repeated trading.

**TEE-capable node — hardware attestation.** A node that runs inference on hardware supporting GPU confidential computing proves, through a hardware-signed attestation, that the designated model actually ran on the designated input, and once confirmed it passes instantly without re-execution. Because the verification cost is structurally low, it receives preferential parameters, whose justification and trade-off are covered in 7.4.

**Ordinary node — optimistic sampling re-execution.** The supply of a node without TEE is accepted optimistically at first, and a verifier randomly selects a portion of the work, re-executes it, and compares. There is a problem specific to LLMs here. Feeding the same input to the same model can produce slightly different output depending on the order of floating-point operations and batching, so reproduction verification that demands bit-for-bit equality does not hold. PoI handles this with a fixed seed and a tolerance: the supply node runs inference with the protocol-designated seed and leaves a log, and the verifier re-executes with the same seed and judges a match if the deviation is within tolerance. Whether this adjudication is actually feasible was tested with real LLMs in Phase 1. In a homogeneous setting where the supply node and the verifier use the same runtime, comparing against the verifier's own regeneration alone separated honest output from four kinds of forgery (weaker-model substitution, early termination, garbage output, and a different prompt) cleanly. In a heterogeneous setting where the two use different runtimes, honest deviation and forgery overlapped somewhat, producing a measurable false-negative rate; there, scoring the log-likelihood of the claimed output proved more robust than comparing a regeneration. The experiment used small models on CPU, so it must be re-validated at scale, but that re-execution adjudication holds in principle is empirically supported (evidence in the implementation repository).

**Dispute — duplicate-execution consensus (second instance).** If the deviation sits on the boundary of tolerance or the judgment is contested, multiple independent verifiers duplicate-execute the same work and finalize by majority consensus (a second instance above the sampling first instance). If forgery is confirmed, slashing is enforced, and a false challenge is paid for out of the challenger's own collateral. The verifiers' own honesty is secured by the same principle: a verifier likewise deposits collateral and is randomly assigned to samples, and if it is confirmed to have passed something without re-execution or to have colluded with a supply node, its collateral is slashed. Because a challenge is open to anyone who has posted collateral, even if the assigned verifier looks the other way, a third party can raise a dispute.

**Roadmap — zkML.** zkML, which turns "inference ran correctly" into a zero-knowledge proof, is the ideal endpoint of this problem, but the current cost of proof generation overwhelms the original computation at LLM-scale inference and is outside the practical range. Axon does not claim zkML as a technology available now, and places it only as a roadmap item to replace sampling re-execution at the point when proof cost enters the practical range.

### 7.3 Verification Economics — Making the Expected Value of Fraud Negative

Because sampling is not exhaustive verification, an individual forgery may slip through the net. The system is nonetheless secure because the goal is not perfect detection but making the expected value of fraud negative. Assuming a rational attacker, the security condition is one line.

```
p × S > (1 − p) × V
```

p is the probability that fraud is detected, S is the amount lost to slashing upon detection, and V is the issuance value gained through fraud. When the expected loss upon detection exceeds the expected gain upon non-detection, forgery does not pay.

The problem is the unit in which this condition is made to hold. Taking each individual issuance transaction as the verification unit walks into a trap. Verifying every transaction lets the overhead erode the issuance value, while verifying only a portion lowers the per-transaction detection probability p, so the S (the bond) needed to meet the condition grows unrealistically large — if p is 1%, S must exceed 99 times V.

PoI raises the unit of verification from the transaction to the epoch. Each epoch, a supply node submits the entire work log of that period to the ledger as a Merkle commitment. The commitment prevents after-the-fact swapping of the log, and the verifier randomly draws k pieces of work from the committed log and re-executes them. Because the sample is drawn using public randomness derived from the ledger after the commitment is finalized, the node cannot know at commitment time which work will be re-executed, and the verifier has no discretion in choosing the sample. Failing to disclose the raw log of a drawn sample within the deadline is itself treated as forgery and slashed, which blocks the evasion of committing and then hiding only the unfavorable parts. On this premise, if a node forged a fraction f of its log, the probability that all k samples miss the forgery is (1−f)^k, so the detection probability in one epoch is `1 − (1−f)^k`.

| Forgery fraction f | k = 20 | k = 30 | k = 50 |
|--------------------|--------|--------|--------|
| 20% | 98.8% | 99.9% | ~100% |
| 10% | 87.8% | 95.8% | 99.5% |
| 5% | 64.2% | 78.5% | 92.3% |
| 1% | 18.2% | 26.0% | 39.5% |

At k=30, fraud with a forgery fraction of 10% or more is detected within a single epoch with probability 95% or higher. What remains is the lower-left of the table, small-scale forgery. The detection probability for f=1% is only 26.0% per epoch, but issuance is a repeated game. Because independent sampling repeats each epoch, the cumulative detection probability of a node that continues forging converges to 1, and even at f=1%, continuing for 10 epochs exceeds 95%. Small-scale forgery is not a hole to slip through but merely fraud whose detection is deferred.

The commit-sample-slash mechanism itself was also prototyped in Phase 1 with real Merkle commitments to verify the plumbing's correctness. Four protocol attacks — post-commit tampering of the log, selective non-disclosure of a sampled entry, forgery of an inclusion proof, and sample evasion (grinding) — all resolved to slashing, and because the public-randomness sample drawn after commit finalization is independent of the placement of forgeries, the measured detection rate over many epochs reproduced the closed form `1 − (1−f)^k` above (on the honest-disclosure path). This prototype is confined to the correctness of the commit, sample, and slash plumbing; the actual ledger derivation of the public randomness that seeds the sample and the empirical measurement of re-execution adjudication are each the province of a separate layer (evidence in the implementation repository).

On the other side of the expected value is slashing. Issuance is not finalized immediately upon commitment but passes through a challenge window (the same device placed on oracle finalization in 4.3), and a node whose forgery is confirmed forfeits its entire bond and the unfinalized issuance within the challenge window. V is limited to the issuance value of the forged fraction, but S is the node's entire stake. As f shrinks the detection probability p falls too, but p falls more slowly than the scale of forgery, so the expected loss per unit of forgery is, if anything, larger the smaller the forgery. Forgery's profit-and-loss does not improve even by shrinking the scale.

### 7.4 Parameters and Cost Accounting

The following is the draft parameter set that fixes the above structure numerically. The values are on a whitepaper-draft basis and are re-validated by simulation at the implementation stage.

| Parameter | Draft value | Rationale |
|-----------|-------------|-----------|
| Epoch length | 1 hour | Balance of issuance-finalization delay vs commitment overhead |
| Sample count k | 30 per node per epoch | Detects f≥10% forgery at 95%+ (7.3 table) |
| Challenge window | 24 hours | Period for challenges before issuance is finalized |
| Minimum bond | 5–10× issuance within the challenge window | Satisfies the security condition (verification below) |
| Verification tax | 3% of new issuance | Self-funds verification cost (5.1·5.4) |
| TEE node preference | Sample count 1/10, bond 1/2 | Verification cost is genuinely lower (below) |

The bond multiple can be verified against the security condition. The upper bound on the V an attacker can target is the issuance receivable through forgery within the challenge window, and if the bond is 5× that, then S ≥ 5V, so p × S > (1 − p) × V holds as long as p exceeds 1/6, about 17%. In the 7.3 table, at k=30 the region where the detection probability falls below 17% is only the extremely small forgery of f below 1%, and in that region the V gained is itself negligible relative to the bond, on top of being closed by cumulative detection. A bond of 5–10×, combined with a sample of k=30, makes the expected value negative at every forgery scale, and the relaxed parameters of TEE nodes (sample 1/10, bond 1/2) are also set within the verification range of the same security condition.

TEE preference is not a privilege but a reflection of cost. A node that passes via attestation induces almost no re-execution verification, so the verification cost the network pays is genuinely low, and the low cost simply returns as a lower sample count and bond. Axon does not mandate TEE. Ordinary GPU nodes can also participate in issuance on the standard path, and the preference induces TEE adoption as a market incentive rather than a mandate. The trade-off is recorded honestly too: because trust in attestation presupposes trust in the hardware vendor's signing scheme, TEE is only one path in the verification stack, not the sole one.

What is the cost of all this verification? The verification overhead, combining sample re-execution and dispute adjudication, is designed to be on the order of 1–3% of the network's total compute, funded by the verification tax foreshadowed in 5.1. New issuance of 1 AXN is divided exhaustively into the supplier's share of 0.97 and the verification tax of 0.03 (5.4), and the verification tax covers both the reward of PoI verifiers and the operation reward of ledger consensus nodes (6.2). The detailed allocation ratio between the two rewards is fixed at the implementation stage, but the funding source itself is fully covered by a single verification tax, and there is no additional issuance beyond the issuance anchor and no external subsidy. That an audit cost is built into currency issuance — this is the principle of the verification tax.

These parameters have passed a first round of validation in a pre-testnet stress simulation for Phase 1. After an agent-based Monte Carlo model reproduced the closed-form detection probability of 7.3, the draft values above kept the attacker's expected value negative under four attacks: rational adaptive forgery, non-stationary and distributed forgery, verifier collusion, and selective disclosure refusal. What the simulation revealed is where the weight of the defense actually rests. The bond is a margin of safety; what makes forgery unprofitable is detection. Because cumulative detection across epochs converges to one, the expected value stayed negative even when the bond multiple was driven to an extreme low. The one point at which the defense broke was when every assigned verifier colluded and third-party rechecking in the challenge window disappeared entirely. What bears the load in this design is not the bond but the random assignment of verifiers and the openness of the challenge window to objection, and the default values above sit far from that breaking point. This result is nonetheless the output of a model with conservative simplifications and does not replace field measurement on the testnet.

This simulation initially assumed that re-execution adjudication is perfect — that it always catches forgery. The heterogeneous-runtime experiment of the previous section disproved that assumption, since the adjudication carries a measured false-negative rate, so we fed that rate (about 0.25–0.375) back into the model and swept again. Even with imperfect adjudication, the parameters above kept the attacker's expected value negative. The reason is a two-layer compensation: a failed adjudication in one epoch is caught by re-execution in later epochs, so cumulative detection across the challenge window still approaches one (trial-level detection at or above 0.997), and once a node is eventually caught, forfeiture of the bond overwhelms the forgery gain. In short, even under the empirically measured condition of imperfect adjudication at heterogeneous runtime, the economic defense of the issuance path holds (evidence in the feedback sweep of the implementation repository).

### 7.5 Issuance Eligibility — What Triggers Issuance

Now we close the boundary Chapter 6 delegated. The compute supply that triggers new issuance of AXN is defined as supply that satisfies all of the following: it must be registered as a supply node with a bond deposited, it must have submitted its supply record as an epoch commitment, it must have passed the verification stack of 7.2, its challenge window must have elapsed without objection, and its supply record must correspond to demand assigned by the network. The last requirement is an axis separate from the authenticity of execution. A node that self-commissions meaningless work and runs genuine inference passes the four preceding requirements, but self-dealing compute with no demand is not eligible for issuance (demand matching is in Chapter 8). Only supply meeting all of these triggers issuance of 1 AXN per 1 ACU along the issuance anchor of 5.1.

Conversely, an ordinary A2A transaction, however much compute it contains, does not trigger issuance. As seen in 6.5, a service payment is a transfer of an existing balance, not new currency, and its verification ends with ① of 7.1 — the buyer's acceptance. The separation of the settlement path and the issuance path (6.5) thus overlaps exactly with the two-track verification (7.1). Payment that flows lightly and issuance that is guarded heavily — the separation of the two paths is the summary of Proof of Inference.

With the rules of the currency (Chapter 5), the ledger on which those rules run (Chapter 6), and the verification that guards the authenticity of issuance (this chapter), the backbone of the system is complete. The next question is who moves on top of this backbone. That is the subject of Chapter 8.

## 8. Ecosystem

The rules of the currency (Chapter 5), the ledger (Chapter 6), and the verification (Chapter 7) built the backbone. This chapter defines the roles and incentives of the five participants that move on top of it, fixes the demand-matching structure that 7.5 delegated, and binds them into a single growth loop (the flywheel).

### 8.1 The Five Participants — Roles and Incentives

| Participant | Role | Incentive to participate |
|-------------|------|--------------------------|
| Agent | Buyer and seller of A2A services | Ultra-small transactions that did not hold up on existing rails (2.1) become payable, so it divides labor more finely and procures and sells more widely |
| Compute-supply node | Supplies verified compute (ACU) to the network | Receives 1 AXN in issuance per 1 ACU supplied (0.97 after the verification tax, 5.4) — direct monetization of held compute resources |
| Verification node | PoI verification (sample re-execution, dispute adjudication) and ledger consensus | Reward proportional to contribution from the verification-tax pool (3% of new issuance) (7.4) |
| Developer | Building and selling agents, tools, and services | A revenue path settled by streaming in proportion to usage through per-LLM-token billing (1.3) rather than seat subscription |
| Enterprise | Operating entity (owner) that delegates a budget to agents | Agent delegation with clear liability, under the spending limit and audit trail of the DID-owner model (6.4) |

A few points deserve elaboration. An agent stands on both the demand and supply sides. It buys the service of another agent for its higher goal while selling its own specialized capability, and most of the ultra-high-frequency, ultra-small transactions of 1.2 occur at this layer. "Verification node" bundles the PoI verifier that guards the issuance path and the L1 ledger consensus node, and the reward funding for both is fully covered by a single verification tax (7.4). The enterprise is the source of final demand. Even if an agent decides on hundreds of purchases per second, that budget was ultimately allocated by the owner — the enterprise (or individual) — and the owner-liability model is the premise for this delegation to hold in a regulatory environment (2.3, Chapter 11).

We also state one boundary explicitly. The only paths by which the protocol supplies AXN are issuance and rewards. The on-ramp of acquiring AXN with fiat is the province of third-party services such as exchanges, outside the protocol's design scope.

### 8.2 Demand Matching — Only Assigned Demand Triggers Issuance

Here we fix the structure of the last requirement 7.5 delegated — that supply must correspond to assigned demand. It has three steps.

1. **Demand posting.** An agent (or its owner) wishing to buy compute posts the work specification, required quality, and willingness to pay to the network's demand queue.
2. **Assignment.** The network's matching rule assigns the posted demand to registered supply nodes on the basis of price, available capacity, and reputation, and leaves an assignment record on the ledger. Because assignment is performed by the protocol and not by the demander, the demander cannot designate a particular supply node, and matching between demand and supply belonging to the same owner is excluded from assignment.
3. **Correspondence check.** In issuance review, a supply node's epoch commitment (7.3) is checked against the ledger's assignment record, and only supply corresponding to assigned demand is recognized as eligible for issuance.

What this structure targets is fraud on an axis different from the authenticity of execution — namely self-dealing. A node that answers self-generated demand with genuine inference passes the verification stack of Chapter 7 without a flaw, but with no corresponding assignment record on the ledger it does not reach issuance. If the verification stack filters out fake supply, demand matching filters out supply with no demand; the two filters are orthogonal, and issuance is permitted only for supply that passes both.

One thing to make clear: the demander's acceptance does not gate issuance. The gate for issuance is, throughout, the verification stack of Chapter 7, so even if a demander and a supply node collude, supply that fails verification does not reach issuance, and acceptance is involved only in ① of 7.1 — the finalization of the transaction payment.

### 8.3 The Flywheel — Demand Calls Supply, Supply Widens Demand

The incentives of the five participants mesh into a single loop.

```
 agent demand rises ──▶ supplier revenue rises ──▶ compute supply expands
        ▲              (1 ACU → 1 AXN issued)          │
        │                                              ▼
 service diversifies ◀── developers/enterprises enter ◀── cost falls via competition
```

When agent demand rises, the issuance receivable from verified supply rises, so supply nodes flow in, and supply competition lowers compute costs. Lowered costs give developers the margin that makes more services viable and enterprises the unit price that justifies wider delegation, and the diversified services in turn raise demand. Each node coincides with the incentives of 8.1, and while the loop turns, the price is held to the value of compute on the arbitrage axis of 5.2, so demand and supply growth do not leak into a surge or plunge in value. Because the verification nodes' reward is proportional to issuance (3% verification tax), the more the flywheel turns, the more the security budget grows alongside it.

The weak point is ignition. In the early stage with neither demand nor supply, no node turns on its own. The supply-side ignition device is the bootstrap reward weighting fixed in 5.4, under which early supply nodes receive a supplementary reward for the same contribution. On the demand side, the operating entity injects the agent workloads of itself and early partner enterprises as the network's first demand, creating the demand that supply nodes can be assigned. Its scale and timing are covered in Chapter 10.

The remaining question is the size of the market on which this flywheel turns, and what the operating entity earns on top of it. That is the subject of Chapter 9.

## 9. Market and Business Model

The flywheel of Chapter 8 is a structure, not a size. This chapter answers the size of the market on which that structure will turn (9.1–9.2), what the protocol and the operating entity earn on it (9.3–9.4), and whom it competes against (9.5).

### 9.1 Market Size — Three Layers of Demand

The market Axon targets is three layers deep. At the innermost is AI inference spending, the real good that gets paid for; outside it, agentic AI spending, the proliferation of the actors that consume the real good; and at the outermost, the whole of agent-mediated commerce. We look from the inside out.

**First layer — AI inference spending.** The AI inference market is projected to grow from USD 106.15 billion in 2025 to USD 254.98 billion in 2030, a 19.2% CAGR[^1]. The shift in composition is steeper still. In 2026, inference workloads are projected to make up about two-thirds of all AI compute, a reversal from one-third in 2023[^2]. The center of gravity of AI spending is moving from one-time training to usage-proportional inference, and inference spending is by its nature entirely recurring payment.

**Second layer — agentic AI spending.** Gartner projects that worldwide agentic AI spending will reach USD 201.9 billion in 2026, up 141% year over year, and overtake chatbot and assistant spending in 2027[^3]. Narrowing to the AI agent market alone, it is projected to grow from USD 7.84 billion in 2025 to USD 52.62 billion in 2030, a 46.3% CAGR[^4]. It is an index of the speed at which the consumer of inference passes from a human's tool to an autonomous agent.

**Third layer — agentic commerce.** McKinsey estimates that agent-mediated commerce (agentic commerce) will orchestrate USD 3–5 trillion of transactions worldwide by 2030[^5]. The shift described qualitatively in Chapter 1 is backed by the quantitative projections of major institutions.

Outside the three layers we place one baseline for comparison: the stablecoin, a precedent for the scale a new payment rail can reach. Total stablecoin transaction volume in 2025 was USD 33 trillion, up 72% year over year[^6], and even on an adjusted basis that strips out non-organic activity such as bots, about USD 9 trillion was settled over the trailing twelve months[^7]. It is a case of a rail that did not exist a decade or so ago reaching an annual settlement volume in the trillions of dollars, and if demand not even premised on machine-to-machine payment reaches this scale, the settlement volume of an economy in which machine-to-machine transactions become the default stacks on top of it.

We do state a limit, however. We could not confirm a reliable institutional source for a quantitative forecast of the machine-to-machine (M2M) micropayment market, and this whitepaper does not cite such a figure. On the structural potential of an M2M economy and the limits of existing infrastructure, the European Central Bank's analysis reaches the same diagnosis as Chapter 2[^8].

### 9.2 TAM — Inference Spending Is Potential Payment Volume

Axon's TAM logic is one sentence. All of AI inference spending is a payment from someone to someone, and the more that payment is reorganized into the A2A ultra-small, high-frequency segment, the more the part that existing rails cannot handle (Chapter 2) becomes Axon's potential payment volume. The protocol's revenue is the fee on that payment volume — that is,

**Annual protocol fee revenue = potential payment volume × penetration rate × fee rate.**

Computing the conservative and aggressive scenarios on a 2030 basis yields the following.

| Input | Conservative scenario (2030) | Aggressive scenario (2030) | Nature of the value |
|-------|------------------------------|----------------------------|---------------------|
| Potential payment volume | USD 255 billion — AI inference spending forecast[^1] | USD 3 trillion — lower bound of the agentic commerce forecast[^5] | Source footnote |
| Penetration rate | 5% | 10% | Assumption |
| Network annual payment volume | USD 12.75 billion | USD 300 billion | Product of the two values above |
| Fee rate | 0.1% — lower bound of the range (9.3) | 0.3% — middle of the range (assumption) | This whitepaper 5.3·9.3 |
| **Annual protocol fee** | **~USD 12.75 million** | **~USD 900 million** | Payment volume × fee rate |

The difference between the two scenarios comes from the definition of what is paid for. The conservative scenario narrows it to inference spending itself and assumes that spending passes through settlement only once (turnover of 1). In reality, an A2A subcontracting chain settles the same final spending at each step, so payment volume exceeds spending; this assumption is conservative on the downside. The aggressive scenario widens it to the whole of agent-mediated commerce, applying a 10% penetration rate to the lower-bound value of the forecast but using not the top of the fee range but the midpoint of 0.3% — so as not to stack all three inputs at the top in a scenario that already takes what is paid for and the penetration rate aggressively. The penetration rate is an unsourced assumption in both scenarios, and where it settles is a question of how fast the flywheel of Chapter 8 turns.

We also record plainly that the conservative scenario's USD 12.75 million per year is a small scale as a standalone business. Half the answer is that the operating entity's funding is not the fee alone but is diversified across the three streams of 9.4, and the other half is the independence of the security budget. Because the verification reward guarding the issuance path comes not from the fee but from the verification tax tied to new issuance (7.4), verification economics does not waver even where fee revenue is small. In addition, a portion of this revenue is the burned share, so the amount attributed to the foundation and operating entity is smaller than the table's value.

### 9.3 Protocol Fee — Rate and Allocation

We fix the rate that 5.3 and 6.5 delegated. The protocol fee is charged on every A2A transaction in the range of **0.1–0.5%** of transaction value. The base rate and transaction-type-specific rates are set within this range through calibration on testnet data (Chapter 10), while the range itself is fixed at the whitepaper stage. Against a card network's 2–3% plus a fixed per-transaction fee (2.1), it is anywhere from single-digit to tens-of-times smaller, and because there is no fixed fee, the fee rate does not diverge even on sub-cent transactions.

Of the collected fee, a portion is burned immediately and removed from circulation (the money-supply-regulation function of 5.3), and the rest becomes revenue for the foundation and the operating entity. Because the allocation ratio between burn and revenue is the balance point between a monetary function (money-supply regulation) and a business function (operating funds), it is set not as a fixed constant but as a governance parameter.

### 9.4 The Operating Entity's Revenue Sources

The revenue sources of the operating entity and the foundation are three.

| Revenue source | Nature | Driver of scale |
|----------------|--------|-----------------|
| ① Share of protocol fees | Recurring revenue | Network payment volume (the scenarios of 9.2) |
| ② Value of the initial-allocation token holding | Capital gain | Price upside in a demand-growth phase + stable value anchored to compute value (see text) |
| ③ Enterprise services (add-on) | Recurring revenue | Enterprise demand for agent-budget operation (8.1) — services on top of the protocol, such as node-operation outsourcing, settlement/audit tools, and spending-control dashboards |

② must be read together with the price anchor of 5.2. In a demand-growth phase, until verified supply catches up with demand, the market price may diverge above the anchor, and that interval is the upside of the holding. But because by design the price converges to compute value over the long run (5.2), the default value of the holding is not unbounded appreciation but a stable value anchored to compute value. Because ① is tied to payment volume, ② to demand growth, and ③ to enterprise adoption, all three point in the same direction — the growth of the network's real usage. The intent of this configuration is to ensure the operating entity has no lever to grow revenue by any means other than growth. The initial allocation ratio and vesting of ② are fixed in Chapter 10.

### 9.5 The Competitive Landscape — The Axis of Differentiation Is Issuance

The approaches that will compete over the problem of A2A payment are four at the category level.

**Crypto payment on general-purpose chains.** Token payment on existing public chains, including stablecoins, overlaps in that there is no account barrier, but in the ultra-small, high-frequency segment it carries the limits of 2.4 unchanged (fees unrelated to transaction value, finality of seconds to minutes, throughput). More fundamentally, issuance is unrelated to compute: stablecoins are anchored to fiat reserves and general-purpose tokens to their own issuance schedule, so the money supply does not incentivize compute supply (5.6).

**Cloud credits.** The compute credits vendors sell are the closest in that they share a scale with compute, but they are not a currency. They are closed within a single vendor, do not presuppose transfer and settlement between third parties, and their value is subordinate to the vendor's pricing policy and credit, so value cannot flow between agents of different owners.

**API aggregators.** Businesses that bundle and resell multiple models and services provide procurement convenience but are not payment infrastructure. Settlement is still on fiat rails, metering is merely a conversion of per-vendor price lists rather than a standard unit, and because the revenue model is the margin, minimizing fees conflicts with their interest.

**Decentralized compute networks.** Decentralized networks that issue tokens tied to the performance of compute or ML work (the so-called DePIN category) are the closest to Axon on the axis of "compute supply triggers token issuance," which is why the distinction matters most. The difference is at the layer of the thing built. These are marketplaces for compute and work, not an A2A settlement currency and ledger. They do not target the four requirements of an ultra-small, high-frequency payment rail (2.5), and the basis for issuance is not the ACU but each network's own scoring, so issuance, metering, and settlement are not integrated on a single unit. A market where you sell compute and receive a token, and a monetary system in which that token meters, settles, and clears compute, are things at different layers.

The axis of differentiation against the four categories is one. **The combination of compute-standard issuance and the verification (PoI) that guards it** — a closed loop in which verified compute supply issues currency in a standard unit and compute is settled with that currency — and none of the four categories combines issuance, metering, settlement, and verification into this single loop. Settlement performance is a point of competition that can converge with time, but the structure by which the money supply procures compute supply for itself (5.6) comes only from a design that builds the issuance rule into the protocol. This axis is Axon's sole axis of differentiation and its line of defense.

Having confirmed the size of the market and the revenue structure, what remains is the allocation of the initial supply and the timetable of execution (Chapter 10).

[^1]: MarketsandMarkets, "AI Inference Market" (2025). <https://www.marketsandmarkets.com/Market-Reports/ai-inference-market-189921964.html>
[^2]: Deloitte forecast. Secondary citation via the Valor C3 blog (2026). <https://valorc3.com/ai-inference-edge-data-center-strategy>
[^3]: Gartner, "Forecast: AI Spending, Worldwide, 2024–2029, 4Q25" (2025-12). A paywalled report; secondary citation via Louis Columbus, Software Strategies Blog (2026-02). <https://softwarestrategiesblog.com/2026/02/16/gartner-forecasts-agentic-ai-overtakes-chatbot-spending-2027>
[^4]: MarketsandMarkets, "AI Agents Market" (2025). <https://www.marketsandmarkets.com/Market-Reports/ai-agents-market-15761548.html>
[^5]: McKinsey & Company, agentic commerce research announcement (2026). <https://www.linkedin.com/posts/mckinsey_our-research-estimates-that-by-2030-agentic-activity-7444669942127374336-MQl5>
[^6]: Artemis Analytics data, Bloomberg reporting (2026-01). <https://www.bloomberg.com/news/articles/2026-01-08/stablecoin-transactions-rose-to-record-33-trillion-led-by-usdc>
[^7]: a16z, "State of Crypto 2025" (2025). Secondary citation via Yahoo Finance reporting (2025). <https://finance.yahoo.com/news/stablecoin-payments-hit-9-trillion-042534119.html>
[^8]: European Central Bank, "A big future for small payments? Micropayments and their impact on the payments ecosystem" (2023-08). <https://www.ecb.europa.eu/pub/pdf/other/ecb.micropaymentsimpactonnpaymentsecosystem202308~bb92cda8ce.en.pdf>

## 10. Token Allocation and Roadmap

This chapter fixes the three things the preceding chapters delegated to it — the ratio and vesting of the initial allocation (9.4), the funding source of the bootstrap supplementary reward (5.4), and the scale and timing of the cold-start demand ignition (8.3) — and lays out the timetable of execution.

### 10.1 Genesis Allocation — One-Time Supply and the Issuance Anchor

At the network's launch, an initial supply (genesis) is issued once. The allocation is as follows.

| Item | Ratio | Lockup / Vesting |
|------|-------|------------------|
| Ecosystem reward pool | 55% | Released in sequence according to a pre-announced decay emission schedule |
| Operating entity and team | 17.5% | 12-month cliff + 36-month linear vesting |
| Investors | 17.5% | 6-month cliff + 24-month linear vesting |
| Foundation reserve | 10% | Executed upon governance approval |

We spell out the nature of each share. The ecosystem reward pool (55%) is the funding source of the bootstrap supplementary reward fixed in 5.4. The upward reward that early supply nodes and ledger-operation nodes receive for the same contribution is paid not through new issuance but out of this pool, released in sequence according to a pre-announced decay schedule and exhausted when the schedule ends. The schedule is fixed once announced, requiring governance approval and re-announcement only where adjustment is unavoidable. The operating entity and team's share unlocks linearly over 36 months after a 12-month cliff, so it cannot be cashed out before long-term performance is demonstrated, and it is the supply the "initial-allocation token holding" of revenue source ② in 9.4 points to. The investors' share follows a shorter 6-month cliff and 24-month linear vesting. The foundation reserve (10%) does not fix its use in advance but requires governance approval for each disbursement, and its principal use is the demand ignition of the next section.

We first answer whether genesis contradicts Chapter 5. The issuance anchor of 5.1 held that only verified compute supply causes new issuance, yet the genesis supply exists without compute supply. The answer is this. What the issuance anchor guards is the rule that growth of the currency moves in step with compute supply, and a pre-announced, finite initial supply is not an exception to that rule but the initial value from which it begins to operate. That this initial value does not erode the rule is backed by three layers of control: genesis is a one-time event that does not repeat, its scale and conditions are announced in advance by this whitepaper, and its circulation is controlled over time by the lockup and vesting of the table above. Every new issuance after launch follows the issuance anchor without exception (5.1).

We also spell out what the allocation ratios mean over time. Because there is no fixed total supply (5.1), the ratios of the table above are the composition at launch, not a permanent equity share, and as long as compute-standard issuance continues, the share of the genesis supply in circulation declines. But this dilution is not an arbitrary evaporation. Because new issuance is tied only to verified compute supply (5.1, 7.5), the dilution of the genesis share moves exactly in step with the growth of compute supply. Neither issuance without growth nor dilution without issuance is possible in this structure. The absolute quantity of genesis is not fixed by this whitepaper; it is a parameter governance will announce before mainnet launch, recorded as an undetermined item in Appendix A.

### 10.2 The Cold Start — Scale and Timing

We answer the question 8.3 delegated — the scale and timing of the ignition that injects the operating entity's and partners' workloads as the first demand.

Stated honestly, a figure for how many ACU to inject in which month cannot be fixed at the whitepaper stage. That value depends on parameters to be re-validated on testnet and on the supply capacity at launch. What the whitepaper fixes is the funding source of the scale and the condition of the timing. Scale is defined by funding source: supply-side ignition (the supplementary reward) is funded by the emission schedule of the 55% ecosystem reward pool, and demand-side ignition (the operating entity's and partners' workloads) by the governance-approved disbursement of the foundation reserve. Timing is defined not by the calendar but by phase-transition criteria. Demand ignition begins with mainnet launch (Phase 2), and when it is switched off is Phase 2's graduation criterion itself. We fix this in the roadmap below.

### 10.3 Roadmap — By Criteria, Not Dates

Axon's roadmap does not use calendar dates. A date, if missed, becomes a cost in trust, and if conditions are sacrificed to keep it, a larger cost. Each phase advances to the next only when it has met its stated exit criteria.

| Phase | Core work | Exit criteria |
|-------|-----------|---------------|
| Phase 0 — Whitepaper and Community (current) | Publishing the whitepaper and design review, forming early partners and community | Design-review feedback incorporated, testnet implementation scope and verification plan fixed |
| Phase 1 — Testnet | Running PoI sampling verification and the L2 channel prototype, simulation re-validation of the 7.4 parameters, fee-rate calibration (9.3) | Verification parameters fixed by simulation and measurement, target detection rate and finalization time reproduced, participation of external supply and verification nodes maintained above a pre-announced threshold |
| Phase 2 — Mainnet | Executing the genesis allocation, starting compute-standard issuance, injecting the operating entity's and partners' initial workloads (the demand ignition of 8.3) | The payment-volume share of the ignition workloads maintained below a pre-announced threshold for a set period — the flywheel becomes self-sustaining |
| Phase 3 — Expansion | Activating the TEE preference path (7.4), zkML research (7.2), jurisdiction-by-jurisdiction regulatory response (Chapter 11) | An ongoing phase — managed by item-specific criteria |

We note two things. First, the simulation re-validation of Phase 1 is the procedure foreshadowed in 7.4: the draft parameters of the issuance path — epoch length, sample count k, bond multiple, challenge window — are re-validated against testnet data and fixed as mainnet values. Second, Phase 2's exit criterion has the end of demand ignition built into it. The operating entity's and partners' workloads injected at mainnet launch are an ignition device, not a permanent one, and Phase 2 graduates only when the share of the ignition workloads in total payment volume falls below the announced threshold and stays there. So that the transition judgment is not left to the operating entity's discretion, each phase's thresholds and judgment period are announced numerically at the close of the immediately preceding phase. The meaning of this criterion is that a network whose ignition never switches off has no right to speak of expansion.

What remains is to set down head-on the risks this plan will face. That is the subject of Chapter 11.

## 11. Risks and Regulation

The last question investors and partners must ask is what can go wrong. This chapter enumerates risks on the three axes of technology, economics, and regulation, and matches each to the mitigation in the body. The principle is one: what has a mitigation is set down with its supporting section, and what lacks a sufficient one is set down as insufficient.

### 11.1 Technical Risks

| Risk | Description | Mitigation |
|------|-------------|------------|
| Verification failure / fraudulent compute | Forgery that obtains issuance for compute never performed | Staking + epoch commitment / sampling re-execution + slashing (7.2–7.4) |
| Conversion-rate oracle manipulation | Inflating the conversion rate to distort the metering of issuance and settlement | Multi-oracle consensus + per-round volatility cap + challenge window (4.3) |
| Channel exit fraud | Submitting a stale balance state to unilaterally settle a channel | L1 dispute-resolution procedure and deposit collateral (6.2–6.3) |
| Non-determinism misjudgment | Floating-point deviation misjudging honest supply as forgery | Fixed seed + tolerance-based judgment + duplicate-execution consensus as a second instance (7.2) |
| L1 consensus security bootstrap | Sybil/collusion targeting the shallow early consensus-node base | Verification-tax-based security budget (6.6, 7.4) + Phase 1 participation threshold (10.3) |

**Verification failure / fraudulent compute.** PoI's goal is not perfect detection but making the expected value of fraud negative (7.3), and whether it holds depends on the parameters. The draft parameters pass the check of the security condition (7.4) but are not yet field-measured, so the residual is managed by the Phase 1 simulation re-validation (10.3).

**Conversion-rate oracle manipulation.** We revisit the economic response 4.3 deferred. If the conversion rate is inflated, the supplier of that model receives more AXN than the effective value warrants, so oracle manipulation can become a bypass that inflates issuance without passing through the verification stack. Consensus among independent multiple oracles removes the single point of manipulation, a per-round volatility cap mechanically caps the gain from a single manipulation, and the challenge window opens a path to detection before finalization. That said, the selection criteria for oracle operators and the collateral design remain undetermined implementation-stage tasks.

**Channel exit fraud.** This is an attack in which a channel counterparty attempts unilateral settlement with a past balance state favorable to it. Because an updated balance carries both parties' signatures and is backed by L1 deposit collateral, the dispute-resolution procedure prioritizes the more recent dually-signed state, and the damage is confined within the deposit collateral (6.3). This latest-state-wins principle was verified in a Phase 1 prototype as preservation of the honest party's share, and the detailed rule of recency determination is fixed at the implementation stage.

**Non-determinism misjudgment.** In a slashing regime a misjudgment is confiscation of property, so the misjudgment rate is itself a risk. Tolerance-based judgment absorbs honest deviation at the first instance, boundary cases are escalated to the duplicate-execution consensus of multiple verifiers (the second instance), and a false challenge is borne by the challenger's collateral (7.2).

**L1 consensus security bootstrap.** As acknowledged in 6.6, the proprietary ledger does not inherit the accumulated security of an existing chain, so in the early stage with few consensus nodes the relative cost of Sybil attack and collusion is low. The mitigation has two axes: the structure by which the verification tax grows the security budget in proportion to issuance volume (6.6, 7.4), and the roadmap that makes external supply and verification nodes' participation exceeding an announced threshold a condition of mainnet entry (10.3). That said, the qualification and stake requirements for consensus nodes remain undetermined items to be fixed at the implementation stage.

### 11.2 Economic Risks

**Early liquidity / participation shortfall.** The flywheel does not turn before ignition (8.3). The mitigation is the whole of the bootstrap design of Chapter 10 — on the supply side, the ecosystem reward pool's supplementary reward; on the demand side, the injection of the operating entity's and partners' workloads. Ignition failure surfaces as a non-satisfaction of the Phase 2 exit criterion, and not forcing expansion while the criterion is unmet is the control built into the roadmap.

**AXN price volatility.** The two-way arbitrage of issuance and burn is a buffer that converges the price to compute value (5.2), not a guarantee of a peg. Because there is neither a reserve nor an institution that posts a price, in a period of abrupt demand shifts the price can diverge from the anchor in the short term. Metering and settlement within the network sit on the ACU scale, so the divergence does not break settlement's arithmetic, but an AXN holder bears the price fluctuation of that period. This whitepaper classifies this not as an eliminated risk but as a structurally buffered risk.

**Compute-supply oligopoly.** If a few large nodes dominate supply, issuance concentrates and the independence premise of assignment (8.2) and verification weakens. The current design has no device that structurally prevents this. The reputation and price competition of the assignment rule and the issuance anchor that keeps entry always open to new suppliers are natural forces for dispersion, but mitigating factors, not a prevention. This whitepaper classifies this item as unresolved, monitors supply concentration as a public metric, and places the design of a response, should the threshold be exceeded, as a roadmap task for governance.

### 11.3 Regulatory Risk — The Owner-Liability Model

**The securities and payment-instrument questions.** Even for a token designed around the real-use functions of metering, settling, and issuing compute, the criteria and conclusions of legal characterization differ by jurisdiction, and the final judgment rests with each jurisdiction's regulators and courts. The question is not confined to securities status. Because AXN is designed as a payment currency, whether payment-instrument, e-money, and funds-transfer regulation applies is a question as direct as securities status, and this too differs by jurisdiction. This whitepaper does not assert AXN's legal character in any jurisdiction, and each stage of issuance and circulation proceeds on the premise of legal counsel in the relevant jurisdiction. This whitepaper is not legal advice and must not be used as a basis for investment decisions.

**The owner-liability model.** The axis of regulatory response is the owner-liability model of 6.4. The fundamental question regulation poses to the agent economy is whom to hold to account for what a machine did, and Axon's answer is built into the structure. No agent DID can participate unless it is linked to a human or legal-entity owner, and legal liability attaches to the owner along the DID. KYC is performed at the owner level, not the agent level — because a verifiable identity is that of a human or a legal entity. The per-transaction-actor audit trail (6.4) provides the basis for after-the-fact accountability, and the spending limits and scope of authority the owner sets provide ex-ante control. The principle — autonomy to the agent, liability to the owner — becomes, from a regulatory standpoint, the answer that there is always a party that can be regulated.

**AML and sanctions compliance.** Anti-money-laundering and sanctions compliance are likewise addressed by owner-level control. Owner identity verification and sanctions screening operate at the entry point, and the per-DID audit trail provides traceability of fund flows. That said, the detailed design that meets jurisdiction-specific requirements — the screening party, the renewal cycle, the reporting scheme — is not fixed at the whitepaper stage and is placed as a roadmap item.

**Connection to the roadmap.** Jurisdiction-by-jurisdiction regulatory response is an ongoing item of Phase 3 (10.3). Regulation is not a gate passed once but a constraint that differs by jurisdiction and changes over time, and Axon handles it as the combination of a structural answer — the owner-liability model — and the operational task of individual response per jurisdiction.

With the system's design (Chapters 3–7), the ecosystem and market (Chapters 8–9), allocation and execution (Chapter 10), and risk (this chapter), the body closes. What remains is the conclusion that summarizes the argument.

## 12. Conclusion

The argument of this whitepaper returns to the declaration of Chapter 3. The natural currency of the agent economy is compute. The marginal cost of the economic acts agents perform is dominated by inference compute, the smallest billing unit has already been decomposed to the LLM token, and the asset that naturally satisfies the three functions of money — unit of account, medium of exchange, store of value — within the agent economy is compute. The body of this paper has shown that this declaration is not a slogan but a verifiable design.

The summary of the design fits in one paragraph. The standard unit ACU, defined as quality-adjusted compute, preserves the meaning of a measure even amid falling compute costs (Chapter 4), and compute-standard issuance — in which the supply of a verified 1 ACU issues 1 AXN — together with fee burn binds the money supply to real supply (Chapter 5). The three-layer ledger of DAG settlement, payment channels, and identity verification handles ultra-small, ultra-high-frequency payments with fees converging to zero in absolute terms and sub-second finality (Chapter 6), and Proof of Inference guards the issuance path by making the expected value of fraudulent issuance negative (Chapter 7). That issuance, metering, settlement, and verification combine into a single closed loop — this is Axon's sole axis of differentiation and its line of defense (9.5).

What remains is execution. Axon is now at Phase 0, and the next stage is the testnet that re-validates the draft parameters by simulation and measurement (Chapter 10). We propose to investors that they review, on the three-layer market (9.1), the revenue structure aligned with real-usage growth (9.4) and accompany this validation; to compute suppliers, that they convert held compute resources into verified supply and become recipients of issuance rewards (8.1); and to developers, that they build agents and services on a revenue path streaming-settled in proportion to usage rather than a per-seat subscription (8.1). The agent economy is already arriving, and what is missing is the currency and payment layer that meters and settles its transactions (Chapter 1). Axon lays the scale and the rails in that empty place.

## Appendix A. Economic Parameter Table

We gather the economic parameters scattered through the body into one table. The draft values of the issuance and verification paths are fixed after re-validation by simulation and measurement on the Phase 1 testnet (7.4, 10.3), while the fee-rate range and the genesis allocation and vesting are values this whitepaper publishes. Items marked undetermined are values this whitepaper does not fix. The draft values of the issuance and verification paths have passed a first round of validation in a pre-testnet stress simulation (7.4); fixing them still awaits field measurement on the testnet.

| Parameter | Value | Defined in |
|-----------|-------|------------|
| Issuance anchor | Verified supply of 1 ACU of compute → 1 AXN new issuance (before verification-tax deduction) | 5.1 |
| Settlement scale | The price of a service metered at 1 ACU = 1 AXN | 5.2 |
| Protocol fee rate | 0.1–0.5% of transaction value — incurred on each transaction within a channel, collected in a lump sum at settlement | 5.3, 6.5, 9.3 |
| Fee allocation | Partly burned + the rest revenue for the foundation and operating entity. The allocation ratio is a governance parameter (undetermined) | 5.3, 9.3 |
| Verification tax | 3% of new issuance — 1 AXN issued is split into 0.97 for the supplier / 0.03 as verification tax, funding the reward of PoI verifiers and ledger consensus nodes | 5.4, 7.4 |
| Epoch length | 1 hour | 7.4 |
| Sample count k | 30 per node per epoch | 7.4 |
| Challenge window | 24 hours | 7.4 |
| Minimum bond | 5–10× issuance within the challenge window | 7.4 |
| TEE node preference | Sample count 1/10, bond 1/2 | 7.4 |
| Verification overhead | On the order of 1–3% of the network's total compute (design target) | 7.4 |
| Genesis allocation | Ecosystem reward pool 55% / operating entity and team 17.5% / investors 17.5% / foundation reserve 10% | 10.1 |
| Lockup and vesting | Ecosystem pool: pre-announced decay emission schedule / operating entity and team: 12-month cliff + 36-month linear / investors: 6-month cliff + 24-month linear / foundation: executed upon governance approval | 10.1 |
| Genesis absolute quantity | Undetermined — announced by governance before mainnet launch | 10.1 |
| Phase-transition threshold | Undetermined — each phase's thresholds and judgment period are announced numerically at the close of the immediately preceding phase | 10.3 |

## Appendix B. Glossary

- **AXN**: Axon Network's currency token. Issued against verified compute supply (5.1), used in the settlement of services metered in ACU.
- **ACU (Agentic Compute Unit)**: The standard compute unit. 1 ACU is 1K LLM tokens of standard-quality inference by the reference model (reference specification) (4.1), and is a unit of measurement, not a currency.
- **Axon Network**: The A2A payment network in which agents settle compute services metered in ACU with AXN. It operates on the three-layer ledger (Chapter 6).
- **LLM token**: The smallest unit that meters the input and output of LLM inference (1.3). It is always written "LLM token" to distinguish it from the currency token AXN.
- **The Compute Standard**: The design that anchors currency to the real good of compute so that new issuance occurs only when verified compute is supplied (3.3).
- **quality-adjusted compute**: Metering based not on the amount of input compute (FLOPs) but on inference output that meets a reference benchmark quality (4.1). It is the core of the ACU definition that keeps the unit's meaning even as compute costs fall.
- **reference specification**: The way of designating the reference model not as a particular commercial model but as a quality band defined by a benchmark (4.1). Any model that meets the specification can serve as the reference model.
- **conversion-rate oracle**: The device that combines the quality coefficient and cost coefficient to compute and publish each model's ACU conversion rate (4.3). It is finalized through multi-oracle consensus and a challenge window.
- **rebasing**: The procedure that, at a generational change of the reference model, fixes the linking coefficient by parallel computation to guarantee the continuity of ACU value (4.4). It follows the chain-linking method of price indices.
- **issuance anchor**: The 1:1 linkage in which the supply of a verified 1 ACU of compute leads to the new issuance of 1 AXN (before verification-tax deduction, 5.1).
- **settlement scale**: The 1:1 linkage of settlement in which the price of a service metered at 1 ACU is 1 AXN (5.2). It sits on the same scale as the issuance anchor, so two-way arbitrage holds.
- **verification tax**: 3% of new issuance (7.4). 1 AXN issued is split into the supplier's share of 0.97 and the verification tax of 0.03, funding the reward of PoI verifiers and ledger consensus nodes.
- **Proof of Inference (PoI)**: The hierarchical verification protocol that proves the actual fulfillment of the compute supply serving as the basis for new issuance (Chapter 7). Through staking, epoch commitment, sampling re-execution, and slashing it makes the expected value of fraud negative.
- **epoch commitment**: The procedure by which a supply node submits to the ledger a Merkle commitment of its entire work log every epoch (draft 1 hour) (7.3). It prevents after-the-fact swapping of the log and serves as the basis for random sampling.
- **challenge window**: The period during which a challenge can be raised before an issuance or oracle computation is finalized (4.3, 7.3). The draft value on the issuance path is 24 hours.
- **slashing**: The sanction that confiscates the entire bond of a node whose forgery is confirmed and the unconfirmed issuance within the challenge window (7.3).
- **TEE (Trusted Execution Environment)**: The confidential-computing environment that proves via hardware attestation that a designated model was executed on a designated input (7.2). A TEE node passes without re-execution and receives preference in sample count and bond.
- **zkML**: The technology that turns the correct execution of inference into a zero-knowledge proof. Because it is impractical at current LLM scale, it is kept only as a roadmap item (7.2).
- **DAG (Directed Acyclic Graph)**: The ledger structure in which, without blocks, each transaction is appended while directly referencing and verifying prior transactions (6.2). It is the basis of Axon's L1 settlement ledger.
- **payment channel**: The L2 structure in which two agents (or groups) deposit balances on L1 to open, exchanging signature-based micropayments (6.3). It is periodically net-settled to L1 as a single net amount.
- **streaming settlement**: The payment method in which balance updates are signed and exchanged through per-second billing, in proportion to fulfillment progress (6.3). The speed of fulfillment is the speed of payment.
- **agent DID (Decentralized Identifier)**: An agent's unique decentralized identity (6.4). It is the party to every channel and transaction and must be linked to an owner identity.
- **owner-liability model**: The account model that assigns autonomy to the agent and legal liability to the owner (human/legal entity) linked to the DID (6.4, 11.3). KYC is performed at the owner level.
- **ecosystem reward pool**: 55% of the genesis allocation (10.1). As the funding source of the bootstrap supplementary reward, it is released in sequence according to a pre-announced decay emission schedule.
- **genesis**: The initial supply issued once at the network's launch (10.1). Its absolute quantity is announced by governance before mainnet launch, and every new issuance thereafter follows only the issuance anchor.

## Team
<!-- PLACEHOLDER: team information to be provided separately — intentional placeholder -->
