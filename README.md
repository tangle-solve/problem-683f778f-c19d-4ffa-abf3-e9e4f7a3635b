<!-- solve-problem:683f778f-c19d-4ffa-abf3-e9e4f7a3635b -->
# Path diversity against adaptive routing adversaries

A [Tangle Solve problem](https://solve.tangle.tools/problems/683f778f-c19d-4ffa-abf3-e9e4f7a3635b).

## What is the problem?

Bitcoin's operation depends on its P2P network: blocks and transactions must propagate quickly and reach all miners, or the network suffers forks, weakened consensus security, and partition-based double-spend attacks. This propagation happens over the Internet, where routing is controlled by BGP, and any AS on a path can hijack (advertise false prefixes), intercept, or blackhole traffic.

The problem: **construct a peer/relay network topology for a blockchain P2P network such that connectivity between honest nodes survives an adaptive routing adversary** — one that observes the overlay, launches BGP hijacks or route intercepts on specific prefixes/ASes, and re-attacks as the overlay reconfigures.

Formal setting:
- Nodes (peers/relays/miners) mapped to ASes; the Internet's AS-level graph is the underlay.
- Adversary controls a set of ASes (or can mount prefix hijacks attracting traffic through them), possibly with per-hijack cost and detection latency; adaptivity means attacks respond to observed relay deployment and peer selection.
- Constraints: partial deployment (only some nodes run the protection, e.g., a subset of relays or modified clients), hidden transit dependencies (two overlay "diverse" paths may share a transit AS invisibly), and realistic AS business relationships (valley-free routing, transit vs. peering).
- Goal: an overlay with provable survivability — e.g., for every honest node pair, at least one path whose underlay realization avoids the adversarial AS set, or bounded block-propagation delay degradation under any feasible attack.

Ambiguity: the census record is informal. The most reasonable formalization is a two-layer (overlay/underlay) survivable-network-design problem with an adaptive adversary; the precise objective (probability of partition vs. worst-case delay inflation) must be fixed by the solver.

## Why does it matter?

Apostolaki et al. showed ~13 ASes can partition Bitcoin today and hijacks can degrade propagation for a large fraction of miners, enabling double spends and consensus failures. SABRE (2019) fixed the "one effective failure point" issue only partially: its relay placement and connections are static and were evaluated against non-adaptive adversaries. A construction with provable survivability under adaptive routing attacks would harden every BGP-dependent PoW network (Bitcoin, Ethereum-class systems, and by extension any latency-sensitive distributed protocol), and would contribute a general result in two-layer survivable network design — a problem relevant far beyond blockchain.

## What's already known?

- **Hijacking Bitcoin (Apostolaki, Zohar, Vanbever, 2017):** measurements showing BGP hijacks can partition Bitcoin (small adversarial AS sets suffice, exploiting the concentration of miners/hosting) and intercept traffic; delay attacks increase centralization. Establishes the threat model and why naive overlay diversity fails.
- **SABRE (Apostolaki et al., 2019):** a protected relay network. Key design choices: relays placed so that connections between them and clients are (a) short (few ASes on path → hard to hijack without collusion) and (b) not interceptable via any single AS on the path, using both the default route and an alternative diverse path. SABRE guarantees connectivity iff a client's AS avoids the adversarial set, but assumes static, non-adaptive attacks and full-ish relay deployment; it does not handle adversaries that adapt to the deployment or peers whose multiple paths share hidden transit.
- **Classic survivable network design** (edge/vertex-disjoint path variants, e.g., via LP rounding) handles single-layer graphs; it does not model the overlay-to-underlay projection where "disjoint" overlay links share ASes, nor BGP policy routing.
- **Interdomain routing security** (RPKI/ROV, BGPsec) is complementary: RPKI deployment is partial and itself an instance of the partial-deployment problem here.

Known shortfalls: no existing construction gives a guarantee against an adversary that re-routes after observing the overlay; no work jointly optimizes peer selection and relay placement under partial deployment with provable underlay-disjointness.

## What would count as solving it?

Any of the following, in increasing strength:
1. **A construction with a proven guarantee:** a relay-placement/peer-selection algorithm such that, for any adversary controlling ≤ k ASes (adaptive, i.e., chosen after seeing the topology), all honest nodes remain connected with block-propagation delay bounded by a stated function of k and the underlay graph — under an explicit model of BGP route selection and hijack reach.
2. **A matching lower bound / impossibility result:** show that with partial deployment fraction f and adversarial AS budget k, any overlay admits a partition (or delay blowup), delimiting when (1) is achievable.
3. **A validated empirical defense:** a topology generator plus simulation over real AS-topology data (CAIDA/RIPE) and real Bitcoin AS mappings, demonstrating survivability against adaptive attack strategies searched by an adversary simulator, with quantified degradation versus SABRE.

A referee would demand: an explicit adversarial model (which hijacks are feasible, detection time, adaptivity granularity), and either a theorem or reproducible experiments against an adversary at least as strong as the 2017 paper's.

## How to start

1. Read Apostolaki–Zohar–Vanbever 2017 (Sections on partition and delay attacks) to internalize why underlay concentration breaks overlay diversity; then SABRE 2019's threat model and guarantee statement — your work must strictly generalize both.
2. Formalize the two-layer model: nodes → ASes, BGP-hijack reachability as the adversary's action set; decide whether to model adaptivity as a repeated game or as a robust (worst-case) placement problem (the latter is more tractable and likely sufficient).
3. Formulate as constrained survivable network design: choose relay set R ⊆ ASes and overlay edges such that projected underlay paths are AS-disjoint for each survivable pair; check hardness (likely NP-hard, so aim for approximation or heuristic + empirical validation).
4. Main obstacle: hidden transit dependencies — you need accurate AS-level path inference (CAIDA AS relationships dataset) and must acknowledge inference error; model it as uncertainty in the graph. A second obstacle: partial deployment means honest-but-unprotected nodes are the weak link; characterize what fraction of hash power must connect to relays.

## Key references

- M. Apostolaki, A. Zohar, L. Vanbever, "Hijacking Bitcoin: Routing Attacks on Cryptocurrencies," IEEE S&P 2017, arXiv:1605.07524. Measures and demonstrates BGP-partition and interception/delay attacks on Bitcoin's network; defines the threat model. https://arxiv.org/abs/1605.07524
- M. Apostolaki, G. Marti, J. Müller, L. Vanbever, "SABRE: Protecting Bitcoin against Routing Attacks," NDSS 2019, arXiv:1808.06254. Designs a relay network with hijack-resistant connection choices and analyzes survivability against non-adaptive AS adversaries. https://arxiv.org/abs/1808.06254
- (For underlay data/model) CAIDA AS Relationships dataset — the standard source for inferring AS-level paths with business relationships. No single paper; see caida.org. (verify current dataset access)
- K. Bentho, A. Gouglidis, et al. on RPKI/ROV partial-deployment effects — representative of interdomain-routing-security work on partial adoption. (verify; no specific canonical citation recalled with confidence)

## Warnings

- Do not treat overlay link diversity as underlay diversity: two direct TCP connections can traverse the same transit AS. This is the core trap the census record's "hidden transit dependencies" flags.
- SABRE's guarantee is conditional on the client's own AS not being adversarial and assumes non-adaptive attacks; any claimed improvement must compare against an adaptive adversary, not the paper's original evaluation.
- BGP path inference is approximate; a "proof" conditioned on inferred paths is only as good as the inference. State this explicitly.
- "Partial deployment" cuts both ways: both relays and clients may be sparse; solve the weakest-link analysis before optimizing placement.
- The census record is dated 2026 and post-2017/2019 literature may exist (e.g., follow-ups on Ethereum network attacks or RPKI deployment effects); do a fresh literature search before claiming novelty. (verify)

---
*`BCGT-000268` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:
1. **A construction with a proven guarantee:** a relay-placement/peer-selection algorithm such that, for any adversary controlling ≤ k ASes (adaptive, i.e., chosen after seeing the topology), all honest nodes remain connected with block-propagation delay bounded by a stated function of k and the underlay graph — under an explicit model of BGP route selection and hijack reach.
2. **A matching lower bound / impossibility result:** show that with partial deployment fraction f and adversarial AS budget k, any overlay admits a partition (or delay blowup), delimiting when (1) is achievable.
3. **A validated empirical defense:** a topology generator plus simulation over real AS-topology data (CAIDA/RIPE) and real Bitcoin AS mappings, demonstrating survivability against adaptive attack strategies searched by an adversary simulator, with quantified degradation versus SABRE.

A referee would demand: an explicit adversarial model (which hijacks are feasible, detection time, adaptivity granularity), and either a theorem or reproducible experiments against an adversary at least as strong as the 2017 paper's.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/683f778f-c19d-4ffa-abf3-e9e4f7a3635b?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000268.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000268)
