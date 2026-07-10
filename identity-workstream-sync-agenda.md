# Agenda notes — Identity workstream sync

**Context:** working assumption that Eva/Richard developed their draft against an older spec snapshot and may not be tracking the v0.1.0 surface in detail. The opening section is partly briefing, not negotiation — get them on the same page about what's locked before any substantive scope conversation.

**Meeting frame:** the property-vs-mechanism layering is the key idea to land. Everything else routes to a clear owner once that's settled.

---

## 0 — Opening: ground on what v0.1.0 actually contains (5 min)

**TL;DR:** Some of the draft's proposals overlap surface that already exists in the locked spec. Worth aligning on the current state before scope discussion so we don't re-litigate solved problems or duplicate primitives.

**What to say:**
> *"Before we go deep on the identity draft, let me quickly walk through what's already in v0.1.0 — there's a reasonable chance some of what you've proposed maps onto existing surface, and we want to scope this conversation around the real gaps rather than re-deriving things that are locked. Five minutes."*

**Pull up Appendix A** — *v0.1.0 quick-reference of identity-adjacent surface.*

---

## 1 — Workstream responsibility model: property vs. mechanism (15 min, lead topic)

**TL;DR:** The cleanest seam between our workstreams runs between security *properties* and the *mechanisms* that satisfy them. Land alignment here first; everything else routes downstream.

**What to say:**
> *"Spec's discipline is to keep vendors from being locked into one mechanism — standards that pick winners early tend to age badly. Identity's discipline is to make sure the security properties actually bite, since properties that are 'deployment-defined' tend not to exist in any deployment that didn't already care. Both are right. The synthesis isn't compromise — it's a separation of layers."*

**Three layers:**

| Layer | Owner | Cadence |
|---|---|---|
| Property-level normativity in the wire spec ("MUST present a verifiable workload identity") | Jointly owned, spec-driven | Slow (it's a contract) |
| Mechanism-level curation (SPIFFE, OIDC, certs, DIDs, attestation envelopes) | Identity-owned registry | Fast (ecosystem-paced) |
| Operational tier guidance (token lifetimes, step-up flows, control-set tiers) | Identity-owned companion docs | Fast, non-normative |

**The positive-sum framing:**
> *"Identity gives up the SPIFFE-specific mandate but wins the underlying battle — identity stops being optional in any meaningful sense. Spec gives up the 'fully permissive wire' but gains a standard that actually means something when claimed."*

**Operating model going forward:**
> *"In the ideal version of this, Identity comes to us with 'we're looking to support functionality A, B, C — how can we do this?' and we either show how it's possible with the existing spec or work through whether new spec changes are needed. Or 'this is hard to do with the existing surface, can we improve it?' and we have a disciplined answer either way. That keeps the spec coherent while letting Identity move at its own pace within it."*

**Pull up Appendix C** — *the three-layer table with concrete examples.*

---

## 2 — Their open questions (§10) — easy wins (5 min)

**TL;DR:** Both of Eva's §10 open items fit naturally into existing pillars. Quick acknowledgement that signals we read their work.

**What to say:**
> *"On your §10 — the OCSF profile for identity events fits naturally under Trace, alongside the existing OTel and OCSF mappings. AgBOM identity fields are an additive concern under Inspect; we already have a Component shape and a document-level agent block that most of your §6.3 list maps onto. Happy to coordinate on either once the layering conversation lands."*

**Pull up Appendix F** for specifics.

---

## 3 — Scope split: what lands when (15 min)

**TL;DR:** Their draft reads as "ship the full model." Need explicit v0.1 / v0.2 / v0.3 tags. With the property-vs-mechanism layering settled, scope becomes a routing exercise.

**What to say:**
> *"With the layered model in mind, scope becomes mostly mechanical. Property-level moves land in v0.1 if they're additive to ACS-Core or join an existing profile; mechanism-level moves go into the `acs-identity` profile and ship on Identity's cadence; operational guidance lives in companion docs and ships whenever Identity is ready."*

**Proposed bucket assignment — pull up Appendix D.**

**Specific bucket calls to make in the meeting:**
- The four ACS-Core property mandates we're proposing (verifiable subject, workload-resolving agent_id, signable response, etc.) — get alignment that these are reasonable and land in v0.1.
- §4 (JIT decision pipeline) and §5 (control tiers) → operational guidance / v0.2 profiles.
- §3.4 (data tokenization) → v0.2.

---

## 4 — Overlapping concerns to resolve (20 min)

Each below has a quick TL;DR. Most resolve cleanly once layering is agreed.

### 4a — Guardian two-layer architecture

**TL;DR:** Their taxonomy treats Guardian as monolithic. Locked design has a deterministic layer + optional LLM-backed agent layer.
**Ask:** *"When you say `Guardian Agent`, do you mean the whole enforcement system or just the verdict-issuing workload? Our internal LLM layer needs its own `model` identity for audit transparency."*

### 4b — Token taint vs. provenance lineage

**TL;DR:** Two parallel mechanisms for "contaminated by untrusted input" — theirs on the token, ours on the data via `derived_from`. Risk of drift.
**Ask:** *"Where should taint live? Our read is that token taint should be **derived from** the provenance graph — coarse-fast-reject at the identity layer, no parallel mechanism that can drift."*

### 4c — Single signing model

**TL;DR:** Their `verdict_signature` reads as a separate signing model from `acs-crypto`'s envelope `signature`. (Caveat we should flag honestly: response envelope doesn't have a `signature` slot today — adding it is real work we agree needs to happen under `acs-crypto`.)
**Ask:** *"Is `verdict_signature` the existing envelope signature populated with the Guardian's workload key, or a new mechanism? We're proposing one signing model — algorithm-agile, with Identity supplying key material."*

### 4d — Policy author identity (subject #11?)

**TL;DR:** Their 10-subject taxonomy omits policy author. We've already foreshadowed a separate `ACS-Policy-Attestation` profile (`conformance.md:65`).
**Ask:** *"Does policy-author identity belong as subject #11 in your taxonomy, or live outside under the policy-attestation profile? Our current direction is the latter."*

### 4e — Step-up = ASK (worth raising even if implicit)

**TL;DR:** Their §4.2 step-up flow is a re-derivation of our §9 ASK + `intent_extension`. Worth naming so the right primitive gets used.
**Ask:** *"Step-up in §4.2 looks like our ASK disposition with `intent_extension` — same trigger set, same approver model, same scope semantics. Are you happy normatively referencing §9 instead of re-defining the flow?"*

### 4f — Naming alignment

**TL;DR:** Their `agent_invocation.id` reads like our `session_id` or `turn_id`. Three identifiers for two concepts unless resolved.
**Ask:** *"How does `agent_invocation.id` relate to our existing `request_id` (per-hook) and `session_id` (per-session)? The lifetime you describe sounds like our `session_id` or possibly `turn_id`."*

---

## 5 — Adoption realism (5 min)

**TL;DR:** Sync with Coding Agents workstream (Almog, Stefano) before locking. Most IDE harnesses don't have OAuth 2.1 + DPoP + SPIFFE today.

**What to say:**
> *"Before any ACS-Identity profile content gets fixed, worth a sync with Coding Agents. They own the realistic-for-IDEs question — Cursor, Claude Code, Codex, Factory. If `acs-identity` is too steep, we want to know now rather than after we've shipped it."*

---

## 6 — Next steps and operating cadence (5 min)

**TL;DR:** Lock the layering, route the topics, set the working model.

**Proposed outcomes:**
1. Property/mechanism/guidance layering accepted as the workstream split.
2. Property-level proposals (the four ACS-Core mandates) drafted by spec workstream with Identity input.
3. `acs-identity` profile structure and principal type registry drafted by Identity workstream.
4. Companion operational guidance docs owned by Identity, lives in `docs/topics/`.
5. Coding Agents sync booked before any profile content freezes.

**Working model:**
> *"For future spec-touching identity proposals, the pattern we'd like to establish: Identity comes with functional needs, we work through whether existing surface supports them, and only when it doesn't do we consider wire changes. That keeps the spec coherent and gives Identity a fast lane for everything that doesn't touch the wire."*

---

---

# Appendices

## Appendix A — v0.1.0 quick-reference of identity-adjacent surface

For grounding the conversation. Anything here is **already locked** in v0.1.0.

| Concern | Where it lives today | Notes |
|---|---|---|
| Human identity on session | `session-start.json:8–16` `user_identity` | `user_id`, `roles`, `authentication_method` |
| Human identity per request | `request-envelope.json:88–95` `metadata.user_context` | Same shape, free-string `authentication_method` |
| Agent workload binding | `metadata.agent_id`, `agent_name` (envelope-level) | Resolves to AgBOM document.agent |
| Per-hook invocation id | `metadata.request_id` (UUID) | Every hook |
| Per-session id | `metadata.session_id` | Every hook |
| Per-turn id | `metadata.turn_id` (SHOULD equal `request_id` of `turnStart`) | All hooks inside a turn |
| Trigger identity | `agent-trigger.json:9–16` `trigger_type` enum + open `trigger_source` | `user_message \| scheduled \| external_event \| a2a_inbound \| system` |
| Platform identity | `metadata.platform`, `platform_version` | Orchestrator identity |
| Tenant | `metadata.tenant_id` (reserved, no v0.1 isolation rules) | |
| Identity descriptor type discriminator | `specification.md:279` — reserved for Identity workstream | `posix_uid \| windows_sid \| oauth_subject \| cert_subject \| ...` |
| Subagent identity / authority derivation | `subagent-start.json` `intent_derivation` enum + `subagent_intent` + `subagent_descriptor` | `inherit_full \| inherit_subset \| derived_from_parent \| fresh` |
| In-session scope expansion | ASK + `ask-details.json:44–72` `intent_extension` (`capabilities`, `scope`, `provenance`) | The locked mechanism for Intent mutation |
| Intent immutability rule | `specification.md:205` | Intent.parsed MUST NOT be modified by runtime LLM |
| Approver identity authentication | `specification.md:213` "Approver authentication is REQUIRED. Guardian MUST verify approver identity against policy." | |
| Guardian evaluator metadata | `response-envelope.json:79–92` `metadata.evaluator`, `model_id`, `evaluator_version`, `confidence` | `deterministic \| agent \| composite` |
| Provenance lineage | `provenance.json` `origin`, `source_id`, `derived_from` | Trust derived in policy by default |
| Component inventory (models, MCP, tools, etc.) | `agbom/component.json` + `agbom/document.json` | Per-deploy + mid-session via `agbom/changed` |
| Request signature | `request-envelope.json:98–127` algorithm-agile signature envelope | HMAC, ECDSA, RSA-PSS, ML-DSA, SLH-DSA, hybrids |
| Conformance profiles | `acs-core`, `acs-trace`, `acs-inspect`, `acs-inspect-dynamic`, `acs-provenance`, `acs-crypto`, `acs-audit` | `conformance.md:9` |
| Policy author attestation (foreshadowed) | `conformance.md:65` `ACS-Policy-Attestation` profile | |

**Two known gaps (be honest about):**
- Response envelope has no `signature` slot today — needs to be added under `acs-crypto` for verdict signing.
- The `Principal` schema at `specification.md:279` is reserved but not defined — this is exactly the Identity workstream's job.

---

## Appendix B — Mapping of their draft to existing v0.1.0 surface

Use to show overlap concretely.

| Their §2.2 block | Already lives at | Net-new vs. overlap |
|---|---|---|
| `human` (`sub`, `idp`, `auth_time`, `amr`) | `metadata.user_context` + `sessionStart.user_identity` | Overlap — structured extension of `authentication_method` |
| `trigger` (`source_id`, `signature`) | `agentTrigger.trigger_source` + envelope `signature` | Overlap |
| `platform` | `metadata.platform`, `platform_version` | Overlap |
| `agent_workload` | `metadata.agent_id` + AgBOM `agent` + components | Overlap |
| `agent_invocation` (`id`, `trace_id`, `delegation_chain`, `tainted`) | `request_id` + `session_id` + `turn_id` + `subagentStart`; tainted ≈ provenance lineage | Overlap; `trace_id` net-new but trivial |
| `model` | AgBOM `component[type=model]` + decision `metadata.model_id` | Overlap; per-step `model_ref` is a real small gap |
| `callee` | `toolCallRequest.tool` + `capability` | Mostly overlap; `sender_constraint_method` claim is net-new |
| `guardian` + `verdict_signature` | `response-envelope metadata.evaluator` + (needs new `signature` slot on response) | Overlap on identity, real gap on signature |
| `data_subjects` | (not present) | Genuinely net-new — worth adding |
| Step-up flow §4.2 | ASK + `intent_extension` | Pure overlap — re-derivation |
| OCSF identity events §10 | `extend_ocsf.md` already maps sessionStart/subagentStart/sessionEnd → 3002 Authentication | Mostly overlap |

---

## Appendix C — The three-layer model in concrete terms

| Layer | Examples of what belongs here | Owner | Cadence |
|---|---|---|---|
| **Property** (wire-normative) | "Verifiable workload identity MUST be presentable per action" / "Response envelopes MUST be signable" / "Approver identity MUST be authenticatable" / "Delegation MUST be audit-recoverable" | Jointly owned; spec-driven | Slow — major version cadence |
| **Mechanism** (Identity-curated registry) | SPIFFE / OIDC subject / cert subject / DID / org PKI as `Principal.type` values; attestation envelope types (sigstore, TPM quote, TEE report); delegation chain encodings (RFC 8693 `act`, JWT-SVID chain, on-behalf-of) | Identity workstream | Fast — registry updates, no spec version |
| **Operational guidance** (non-normative companion docs) | Token lifetime ceilings; step-up trigger lists; control-set tier ladders (0/1/2/3); RAR scope patterns; sender-constraint best practices | Identity workstream | Fast — docs ship whenever ready |

---

## Appendix D — Concrete bucket assignment

| Bucket | What it contains | Profile |
|---|---|---|
| **ACS-Core properties** (v0.1) | Verifiable subject in user_context; workload-resolving agent_id; signable response envelope (slot present); approver identity verifiable | ACS-Core |
| **ACS-Identity profile** (v0.1, opt-in) | Full Principal vocabulary; delegation chains; sender-constraint method claim; attestation freshness; data_subjects block; per-step model_ref | `acs-identity` (new) |
| **ACS-Identity-Attested** (v0.1 or v0.2, opt-in sub-profile) | Workload + (when present) runtime carry verifiable attestation | `acs-identity-attested` (new) |
| **Operational guidance** (non-normative) | Token lifetime ceilings; control-set tiers 0-3; step-up trigger lists; RAR examples | Companion docs in `docs/topics/` |
| **Deferred to v0.2** | §3.4 data tokenization; §4 full JIT decision pipeline as spec; A2A outbound identity | v0.2 |
| **Always conditional** | `data_subjects` (only when PII in scope); `trigger` block (only when non-user-initiated) | n/a |

---

## Appendix E — Footgun analysis (motivates the ACS-Core property mandates)

"What could a malicious or sloppy implementer do today and still pass conformance?"

| Footgun in v0.1.0 today | Closed by which property mandate |
|---|---|
| `metadata.user_context.user_id` is a free string. Any agent can populate any user_id. | "Subject MUST be verifiable" — mechanism deployment-defined |
| One static service account per agent deployment is fully conformant. | "agent_id MUST resolve to a workload identity, not a free label" |
| Bearer tokens in agent memory leak via prompt injection. | "Tokens held across calls MUST be sender-constrained" (mechanism: any registered binding) |
| Autonomous-run triggers can claim any source. | "Trigger MUST be authenticatable when non-user-initiated" |
| Guardian verdicts aren't signed. | "Response envelopes MUST be signable" — add slot in `acs-crypto` |
| Long-lived API keys are perfectly conformant. | Operational guidance (Identity-owned), with `acs-identity` enforcing claim-level lifetime metadata |
| `subagent_intent` can claim any capability; only policy stops it. | Use existing `intent_derivation` enum + delegation-chain claim under `acs-identity` |

This is the evidence base for "property-level normativity matters" — and it's also the test for what belongs in ACS-Core vs. `acs-identity` (Core gets the verifiability properties; the profile gets the richer claim vocabulary).

---

## Appendix F — Reframe of their §10 open questions

| Their §10 item | Where it routes |
|---|---|
| "Define a normative OCSF profile for agent identity events at `spec/trace/identity_events.md`" | Lives in **Trace** pillar. `extend_ocsf.md` already maps `sessionStart`/`subagentStart`/`sessionEnd` → OCSF 3002 Authentication and decisions → 2004 Detection Finding. Identity-specific event class additions would extend that mapping; same file, same pattern. |
| "Define the Agent Bill of Materials identity fields formally (§6.3) under `spec/inspect/`" | Lives in **Inspect** pillar. `agbom/component.json` and `agbom/document.json` already define `agent` document-level fields and component types. Identity additions (`guardian_binding`, `trust_roots`, `token_endpoints`, attestation refs) are additive — Identity proposes, Inspect reviews and merges. |

Both items demonstrate the working model in action: Identity surfaces a need; spec routes it to the correct pillar; implementation work happens with the right owner.
