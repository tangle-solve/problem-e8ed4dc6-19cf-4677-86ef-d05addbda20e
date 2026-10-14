<!-- solve-problem:e8ed4dc6-19cf-4677-86ef-d05addbda20e -->
# Tenfold payment-channel capital efficiency

A [Tangle Solve problem](https://solve.tangle.tools/problems/e8ed4dc6-19cf-4677-86ef-d05addbda20e).

## What is the problem?

In a payment-channel network (PCN) like Lightning, participants lock coins into channels; a payment of value *v* routed over a path consumes *v* of balance on each channel along that path. The efficiency metric here is **completed payment volume per unit of locked capital**, i.e., total successfully routed value over a time window divided by total coins committed to channels, while holding constant: (a) safety (atomic, no loss beyond collateral), (b) latency (routing/settlement time), (c) churn robustness (performance under nodes going offline / topology change).

The target: demonstrate a **10× improvement in this volume-per-locked-coin ratio** over the state-of-the-art baseline (essentially Spider-style congestion-aware routing on a fixed topology), evaluated on **published asymmetric workloads** — transaction streams with skewed senders/receivers and non-uniform values, as in the Ripple/ Lightning datasets used in the PCN literature.

Formally: given a graph G = (V, E), channel capacities {c_e}, a stochastic arrival process of payments {r_i = (s_i, t_i, v_i, δ_i)} with deadline δ_i, design a routing + rebalancing + (optionally) capital-allocation policy π such that throughput(G, c, π, workload) is ≥ 10× the baseline throughput under the same total capital Σc_e, with matching payment success ratio and latency, measured via simulation or analysis. The capital may be reallocated over time (rebalancing, channel opening/closing is part of the design space, though churn cost counts against you).

Ambiguity note: "tenfold" is relative to an unstated baseline; the most defensible reading is Spider (Sivaraman et al. 2020) on its own workloads.

## Why does it matter?

Noncustodial payment networks are capital-inefficient: merchants receiving payments drain inbound liquidity and must rebalance, and routing nodes must pre-lock capital in unbalanced positions. A 10× improvement in volume per locked coin would make Lightning-style networks competitive with custodial systems for working capital, directly lowering the cost of running routing nodes and of receiving payments. It also connects to classical inventory-pooling and backpressure theory, so a solution likely generalizes beyond blockchains.

## What's already known?

- **Revive** (Khalil & Gervais 2017): off-chain rebalancing via fee-balanced cycles — detects imbalanced channels and rebalances within the PCN without on-chain transactions. Recovers capacity but is reactive and doesn't optimize aggregate capital allocation.
- **Spider** (Sivaraman et al. 2020, NSDI): payment-channel routing as congestion control — packetizes payments, prices channels by congestion and value in flight, achieves near-optimal throughput on workloads derived from Ripple and Lightning data. This is the canonical baseline; it optimizes throughput given capital, not capital efficiency.
- Later work on channel placement/funding strategies (e.g., heuristic fund-allocation papers) improves utilization modestly, typically well under 2×. (verify — survey before claiming exact factors.)
- Related theory: inventory pooling (aggregation reduces safety stock ~√n), throughput-optimal backpressure routing (Tassiulas–Epomidis-style max-weight), and rebalancing-as-network-flow formulations.

Why these fall short: none jointly optimize dynamic capital reallocation with routing under asymmetric demand; rebalancing literature fixes a topology, congestion-control literature fixes capacities.

## What would count as solving it?

Any of:
1. **Construction + experiment:** a routing/rebalancing policy with a public simulation on published workloads (e.g., Spider's Ripple-derived traces) showing ≥10× completed volume per locked coin vs. Spider under matched total capital, with success ratio and latency within, say, 10% of baseline, plus stress tests under node churn.
2. **Analysis:** a theorem that a class of policies achieves volume-per-coin within constant factor of an offline-optimal LP/flow benchmark that itself is ≥10× the static-allocation baseline under a characterized demand model (e.g., known asymmetric stationary demand).
3. **Lower bound (partial credit):** proof that under a given demand model no policy can exceed k× for k < 10 — this would refute the moonshot and is itself publishable.

## How to start

1. Read Sivaraman et al. (2020) §5–7 (workloads, evaluation, congestion pricing) and set up their simulator as the baseline; compute the baseline volume-per-coin metric exactly — this defines "10×."
2. Read Revive (Khalil & Gervais 2017) for cycle-based rebalancing; ask where it loses capital efficiency (frequency? cycle finding? no joint optimization with routing?).
3. Formulate the offline relaxation: max throughput subject to total capital, over joint channel capacities and flows — an LP/convex program. The gap between this optimum and Spider quantifies headroom; if headroom < 10× on the workloads, pivot to the lower-bound variant.
4. Main obstacle: asymmetric workloads inherently strand capital on the receiving side; the plausible win is dynamic/spliced capital reallocation or virtual-channel constructs that pool liquidity — check that added mechanisms don't violate the latency/safety constraints.

## Key references

- Sivaraman, et al. "High Throughput Cryptocurrency Routing in Payment Channel Networks." NSDI 2020. https://www.usenix.org/system/files/nsdi20-paper-sivaraman.pdf — Introduces Spider: congestion-aware, packetized routing achieving near-optimal throughput on Ripple/Lightning-derived workloads; the baseline.
- Khalil, R., Gervais, A. "Revive: Rebalancing Off-Blockchain Payment Networks." IACR ePrint 2017/823. https://eprint.iacr.org/2017/823.pdf — Off-chain cycle-based rebalancing of depleted channels without on-chain transactions.
- (verify) Flash / SilentWhispers / landmark-routing PCN papers for additional workload datasets and routing baselines.

## Warnings

- The 10× figure is a moonshot target, not derived from any stated bound; the honest baseline and workload choice matters enormously — cherry-picked workloads can trivially inflate ratios. Pre-register the comparison against Spider on its own traces.
- Watch for hidden capital: "rebalancing" payments themselves consume channel balance and fees; protocols that shuffle value in loops can look efficient while spending throughput on self-transfers.
- Latency and safety constraints rule out trivial wins like massive over-provisioning of virtual channels or long settlement delays.
- The census record is AI-generated (fleet source) and may not correspond to a formal open problem in the literature; treat it as an engineering moonshot, and verify the related-work claims above marked (verify).

---
*`BCGT-000176` · unverified*

## Acceptance guidance

Any of:
1. **Construction + experiment:** a routing/rebalancing policy with a public simulation on published workloads (e.g., Spider's Ripple-derived traces) showing ≥10× completed volume per locked coin vs. Spider under matched total capital, with success ratio and latency within, say, 10% of baseline, plus stress tests under node churn.
2. **Analysis:** a theorem that a class of policies achieves volume-per-coin within constant factor of an offline-optimal LP/flow benchmark that itself is ≥10× the static-allocation baseline under a characterized demand model (e.g., known asymmetric stationary demand).
3. **Lower bound (partial credit):** proof that under a given demand model no policy can exceed k× for k < 10 — this would refute the moonshot and is itself publishable.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/e8ed4dc6-19cf-4677-86ef-d05addbda20e?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000176.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000176)
