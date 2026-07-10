# Draft reply on the Microsoft AGT comparison

Thanks for digging this deep. The steelmanning sharpened our case more than the first pass did, so it's worth a careful reply. One of your softening reads doesn't survive the actual schemas; the other two confirm our edge. Reframe first, then the points.

The platform is real, but it's the wrong unit of comparison. Almost all of it sits around the policy contract, not inside it. What decides whether the ecosystem is rich is the control contract: what the policy input can represent, and where it can fire. That surface is much smaller and more opinionated than the platform wrapped around it. We shouldn't fight a platform-breadth contest. We'd lose it, and Fred's right that it isn't the point. We should contest the contract.

## 1. Compositional risk: the gap looks narrower

I read it the other way now, and the audit-chain half actually cuts against the point.

"Stateless, so the policy core is blind to compositional risk" is confirmed. The runtime MUST NOT retain mutable state that influences a verdict, and their own LIMITATIONS.md §1 admits it does not correlate sequences of individually-allowed actions. No dispute.

"The platform carries session state in the transport and the audit chain" is partly true, but mislocated. Per their spec the stateful component is the host: "The host is the stateful policy enforcement point. The host MUST track provenance and labels." The transport doesn't carry session state, it ferries a snapshot the host assembled. The Merkle audit chain (ADR-0017) is forensic tamper-evidence, not a decision-time input. Nothing reads it to compute a verdict. So "state lives in transport and audit chain" is loose. State lives in host code, and only the slice the host puts in each snapshot ever reaches a policy.

"So the gap looks narrower" is where the inference breaks. It conflates "session state exists somewhere in the platform" with "compositional risk is governable." Those are different claims, and the difference is the whole game. Governing compositional risk means intervening on the next action given the trajectory so far: at decision time, at the right control point, portably. State that sits where no decision reads it narrows nothing about governance.

Break "narrower" across the three layers and only one moves:

| Layer | Does platform state narrow the gap? |
|---|---|
| Data representability (can the trajectory be carried?) | Yes, genuinely narrower. The snapshot is open (`additionalProperties: true`) with `prior_decisions`/`messages`. This is the real kernel of the point. |
| Control point (is there a place to evaluate it?) | No, unchanged. Closed 8-point enum; §22 forbids adding one. No turn, retrieval, memory, or subagent point. Even a fully reconstructed trajectory can only be read at a tool call. |
| Engine state and shipped capability (does anything actually do it?) | No, unchanged. The stateless runtime can't accumulate, the accumulator is unshipped host glue, the stock libraries do single-action evaluation, and the trajectory layer is a proposed community extension. |

And the kind of state you're citing cuts against the point. The audit chain is the wrong evidence--a forensic chain you consult after the fact narrows the detection gap, not the governance gap. It tells you a slow-roll exfiltration happened; it doesn't stop the next step. That's the Trace versus Instrument distinction ACS itself draws: observability is not enforcement. Citing the audit chain to narrow an enforcement gap is a category error.

Bottom line: the gap is narrower in one sense, theoretical data-carrying capacity, and unchanged in the two senses that decide whether you can actually govern compositional risk, namely a standardized control point and a shipped stateful evaluator. For any vendor who isn't going to build Microsoft's entire missing trajectory engine out of band, the governance gap is as wide as it first looked.

## 2. Provenance

Their information flow is a flat, single-axis `source_labels: [string]` array. It has no lineage edges and no integrity axis, and "declassif" doesn't appear anywhere in their repo. The runtime stores no labels and propagates no taint. Ours carries transitive lineage as a first-class, normative field: a value's lineage is the union of its inputs, and summarization and compaction count as derivations. The `preCompact` guard turns compaction into a controlled chokepoint instead of a laundering path. Ironically, FIDES, which is Microsoft Research's own paradigm, is absent from their repo and can't be expressed in that contract. One fairness note so we don't overclaim: their label lattice is vendor-substitutable, since they ship `_with_lattice` helpers, so don't say "fixed taxonomy." Say "flat, single-axis, no lineage, no declassification."

## 3. AgBOM

Their "Decision BOM" is reconstructed after the fact (ADR-0018), and they also ship an immutable deploy-time Agent SBOM. Neither one is a live, mid-session inventory that tracks MCP servers, memory stores, knowledge bases, or A2A peers as components with provenance. One honesty flag: our dynamic AgBOM is v2 roadmap, not shipped yet either, so it's ours by design intent, not ours running in production today. It's still the cleanest daylight between the two specs.

## Net

Their policy engine is more pluggable than our first pass assumed, so "they lock vendors in" was too strong and we should drop it. But the contract-level story is ours. Open data, closed hooks: IBAC, FIDES, CaMeL, AARM, and compositional risk all get pushed into un-standardized, non-portable host glue that their own libraries can't evaluate. That's Fred's ecosystem-richness point made concrete. Boiled down to a sentence: their contract governs a single action, ours governs the agent across a session. I'd carry that, plus the vendor-neutrality contrast, into the Microsoft conversation as a complementary pitch: their shipped engine, our paradigm-neutral control contract. Happy to share the source-quoted breakdown for any claim here.