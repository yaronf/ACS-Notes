# Documentation review notes

Working notes from reviewing the local docs preview (`http://127.0.0.1:8000/`).
Each `##` heading is one comment; titles are issue-ready.

**Open-issue check:** Before keeping a comment, search open issues on [`GenAI-Security-Project/agent-control-standard`](https://github.com/GenAI-Security-Project/agent-control-standard/issues). Drop the comment if an open issue fully covers it (same ask). Related-but-narrower issues do not count as duplicates.

Checked against open issues (2026-10-08): no comment fully duplicates an open issue. Closest overlaps (kept): large-payload note ↔ [#155](https://github.com/GenAI-Security-Project/agent-control-standard/issues/155) (truncation only); Approver/ASK clarity ↔ [#175](https://github.com/GenAI-Security-Project/agent-control-standard/issues/175) / [#51](https://github.com/GenAI-Security-Project/agent-control-standard/issues/51); MODIFY/ASK on handshake ↔ [#147](https://github.com/GenAI-Security-Project/agent-control-standard/issues/147) (substitution representation, not ClientHello declaration); scope/audience ↔ [#133](https://github.com/GenAI-Security-Project/agent-control-standard/issues/133) (FAQ page, not agent-class scope).

## Top 5

Curated as we go; re-rank when a new finding outranks an entry below.

1. [Define ACS scope and target audience across agent classes](#define-acs-scope-and-target-audience-across-agent-classes) — who the standard is for is never stated.
2. [Call the HMAC baseline a MAC, and justify why not ECDSA-P256](#call-the-hmac-baseline-a-mac-and-justify-why-not-ecdsa-p256) — Core integrity floor is misnamed and under-justified.
3. [Justify hook negotiation / out-of-band capability vs normative emit-when-applicable](#justify-hook-negotiation--out-of-band-capability-vs-normative-emit-when-applicable) — Core should not need out-of-band "which hooks are live" tables.
4. [Put MODIFY (and ASK) capability on the handshake wire](#put-modify-and-ask-capability-on-the-handshake-wire) — same pattern: capability off the wire / out-of-band.
5. [Clarify what control paradigms mean for implementers](#clarify-what-control-paradigms-mean-for-implementers) — IBAC/FIDES/CaMeL/AARM read like a product matrix, not optional citation styles.

---

## Add A2A hook pages to the MkDocs nav

**Where:** `mkdocs.yml` nav; source pages under `docs/spec/instrument/a2a/hooks/`

**Notes:** MkDocs build warns that these pages exist but are not in nav, so they are hard to reach from the site:

- `cancel_task_request.md`
- `get_task_push_notification_config_request.md`
- `get_task_request.md`
- `resubscribe_to_task_request.md`
- `send_message_request.md`
- `set_task_push_notification_config_request.md`
- `stream_message_request.md`

---

## Rename the Instrument "ACS specification" nav label

**Where:** `mkdocs.yml` → Specification → Instrument → `ACS specification` (`spec/instrument/specification.md`)

**Notes:** "ACS specification" sits two levels under the top-level "Specification" section, which reads as recursive and confusing. Rename the leaf (e.g. to match the page's role under Instrument: protocol/hooks contract) so the nav path is not Specification → … → ACS specification.

---

## Define ACS scope and target audience across agent classes

**Where:** Top-level docs (landing / About / Topics intro); no explicit audience statement found

**Notes:** "Agents" in ACS can mean several very different deployments: enterprise production software agents, coding agents, and personal agents of the Claw ilk. Readers cannot tell which class the standard is written for, or whether all three are in scope. Add an explicit scope and target-audience statement that names these classes and what ACS claims to cover for each.

Checked: `docs/README.md`, `docs/about.md`, and pillar intros frame ACS around enterprises ("the enterprise that welcomes them in", "Enterprises need to see…") but never enumerate agent classes or say whether coding agents or personal agents are in or out of scope.

On ASK: mandatory ASK does **not** by itself make coding agents the primary scope. Guardians MUST support ASK, but approvers MAY be human, agent, or service (§9), and §9.2 substitutes DEFER/DENY for approver-incapable clients (explicitly including headless automation, not only IDE plugins). So interactive coding agents are a natural ASK consumer, not the declared primary audience — but the tension between enterprise framing and interactive-approval gravity is another reason the scope statement is missing.

---

## Balance Inspect transparency against attacker reconnaissance

**Where:** `docs/README.md` — Trustworthy Agents blurb, Inspectable paragraph ("What are the services behind them, and which data they can access.")

**Notes:** The blurb presents that visibility as purely desirable. Too much transparency can also help an attacker map services and data access. The intro should acknowledge the tradeoff, not only the benefit.

---

## Call the HMAC baseline a MAC, and justify why not ECDSA-P256

**Where:** `docs/acs.md` ("baseline HMAC-SHA256 signature"); also `docs/spec/conformance.md`, `docs/spec/instrument/specification.md` §10 (registry correctly labels HMAC as "Symmetric MAC" but the surrounding prose still says "signature")

**Notes:** Formally HMAC-SHA256 is a MAC, not a signature. Calling the Core baseline a "signature" blurs that it cannot provide non-repudiation or bind a compromised Guardian (which the same § already admits). Separately: why is shared-secret HMAC the mandatory baseline rather than a public-key algorithm — ECDSA-P256 being the natural classical choice, already OPTIONAL in the §10.1 registry — with HMAC as an optional same-host shortcut? The adoption/simplicity rationale should be explicit if HMAC stays Core.

---

## Mention ECDSA-P256 in the ACS-Crypto overview blurb

**Where:** `docs/acs.md` — ACS-Crypto profile bullet ("HMAC-SHA256 baseline, ML-DSA-65 / SLH-DSA-128s for PQC, hybrid composites…")

**Notes:** The overview skips classical asymmetric entirely. `ECDSA-P256` is in the §10.1 registry (OPTIONAL, strongest current ecosystem support) and appears only inside hybrid composites in the blurb. A reader scanning the profile list would think ACS-Crypto is HMAC + PQC only. Name ECDSA-P256 (and optionally RSA-PSS) alongside the PQC algorithms, especially given the open question of whether P-256 should be baseline rather than HMAC.

---

## Clarify how Runtime Hooks relate to ACSInstrument

**Where:** `docs/acs.md` — Trustworthy agents are › Instrumentable › "Runtime Hooks" example (follows the earlier "Init agent with ACS" / `ACSInstrument` snippet)

**Notes:** Confusing for newcomers. After wrapping with `ACSInstrument`, this section shows hand-registered `toolCallRequest` hooks that call `guardian.evaluate` themselves. Unclear whether that manual wiring is always required, or only when you need tighter control than `ACSInstrument` already provides. Spell out the relationship (what the wrapper does by default vs when you register hooks).

Same Python snippet only branches on `DENY` and `MODIFY`. ACS dispositions also include `ALLOW`, `ASK`, and `DEFER`, and there is no catch-all path for an unexpected `action`. (Python `if`/`elif` uses `else` for that; `match`/`case` uses `case _:`.) The TypeScript tab's `switch` has the same gap.

---

## Clarify that AgBOM serializations are not required to be signed

**Where:** Inspect definition — `docs/spec/inspect/README.md` (and related CycloneDX/SPDX/SWID extension pages)

**Notes:** Signing a BOM is optional in CycloneDX. The ACS-Inspect profile should state that a signed serialization is not required for conformance. Integrity for Inspect remains the ACS envelope on `agbom/snapshot`/`changed` and the SessionContext chain.

---

## Put MODIFY (and ASK) capability on the handshake wire

**Where:** `docs/spec/instrument/specification.md` §6.5 (MODIFY-incapable clients); same pattern in §9.2 (approver-incapable clients); contrast §4 Capability Negotiation Handshake

**Notes:** §6.5 lists out-of-band ways to learn that a client cannot apply MODIFY (agent identity, `agent_id` policy, org config, "any other signal") and admits ACS does not put the declaration on the wire in v0.1. Those are all unsatisfactory once ACS-Core already requires a capability-negotiation handshake (`methods_implemented`, `profiles_supported`, etc.). Disposition-handling capability (at least MODIFY; same for ASK in §9.2) belongs in ClientHello rather than the Guardian's policy bundle.

---

## Rename the Instrument pillar to name what it does

**Where:** Pillar naming across the docs (e.g. Instrument / Trace / Inspect); `docs/spec/instrument/`, nav, conformance, intros

**Notes:** The three pillars are Instrument, Trace, and Inspect. Trace and Inspect name outcomes; Instrument names the mechanism (hooks) rather than the job (runtime permit/deny/modify of agent actions). Prefer a verb that says what it does — e.g. Govern, Police, or Control — so the triad is parallel.

---

## Include SPIFFE (and future WIMSE) in identity type examples

**Where:** `docs/concepts/identity.md`, `docs/topics/core_concepts.md`, `docs/spec/instrument/specification.md` §11 (identity descriptor `type` examples: `posix_uid`, `oauth_subject`, `cert_subject`, …)

**Notes:** The example discriminator list skips workload identity. SPIFFE IDs (and, later, WIMSE identifiers) are the natural types for agent/workload principals, and `docs/identity/standards.md` already proposes WIMSE/SPIFFE as the recommended model. Add `spiffe_id` (and a forward pointer to WIMSE) to the illustrative `type` set so the core identity text matches the identity workstream.

---

## Consider RFC 9493 Subject Identifiers instead of ad-hoc oauth_subject

**Where:** `docs/concepts/identity.md` (and related identity type examples); Identity workstream descriptor schema

**Notes:** SECEVENT published [RFC 9493](https://www.rfc-editor.org/rfc/rfc9493.html) (Subject Identifiers for Security Event Tokens): typed JSON subjects with a `format` discriminator (`iss_sub`, `email`, `uri`, `did`, `opaque`, …) and an IANA registry. `iss_sub` is a better stand-in for OAuth/OIDC principals than ACS's illustrative `oauth_subject` + bare `sub` (issuer+subject is what makes the id unique). Adopted mainly via OpenID Shared Signals / CAEP / RISC (e.g. Okta SSF); not a universal OAuth wire replacement yet. Worth aligning ACS identity descriptors with that registry (and using `uri`/`did` for SPIFFE/WIMSE) rather than growing a parallel type list.

---

## Identity "verified credential" claim has no ACS wire support (e.g. PoP)

**Where:** `docs/concepts/identity.md` ("An asserted identity is weaker than one bound by a verified credential or signature"); related trust-basis language in `docs/topics/core_concepts.md`

**Notes:** The sentence implies ACS can distinguish asserted vs credential-bound identities. On the wire, identity descriptors are just typed claims; there is no PoP / DPoP / mTLS-bound / SVID presentation field, and auth remains deployment-defined (§ identity + handshake). ACS-Crypto attests *envelopes*, not principal credentials. Identity workstream prose requires DPoP/mTLS compositionally (`docs/identity/standards.md`), but that is not expressed in the Core identity descriptor model. Either add how a Guardian learns that an identity was PoP-verified, or soften the claim so it does not read as an ACS feature.

---

## Replace leftover "ASOP" references with ACS

**Where:** `docs/topics/core_concepts.md` diagram (`docs/assets/agent_env.png` — edge labeled ASOP between Observed Agent and Guardian); `docs/spec/instrument/a2a/extend_a2a.md` ("uses ASOP as a transport"); `docs/spec/trace/OCSF/implementation_examples.md` (`ASOP Security Layer`, `unmapped.asop.*`)

**Notes:** "ASOP" is never defined in the current documentation. It reads as a stale project name (alongside historical AOS branding) left in examples and the Core Concepts environment diagram. Rename to ACS (and `unmapped.acs.*` if that namespace is kept) so newcomers are not hunting for a fourth acronym.

---

## ACS-in-action sequence: DENY is legal at every allow, and AgBOM is mis-ordered

**Where:** `docs/topics/ACS_in_action_example.md` — Sequence mermaid

**Notes:** The walkthrough correctly shows `allow` for a permit path, but a reader can take that as the only legal verdict. Per hooks, DENY is allowed at each decision shown: `agbom/snapshot` (banned component), `sessionStart` (refuse session), `userMessage` (ALLOW/DENY/MODIFY), `toolCallRequest` (full vocabulary), `toolCallResult` (ALLOW/DENY/MODIFY). Call that out (note or alternate deny branch), especially since the page later shows deny-shaped examples only for the tool-call paradigms.

Also: the diagram emits `agbom/snapshot` *before* `sessionStart`. Normative ordering is after `sessionStart` and before the first content-bearing hook (`hooks.md` / Inspect README).

---

## Specify large-payload handling: stream, truncate, compress, or refuse — with an indicator

**Where:** Wire format / Instrument — `docs/spec/instrument/specification.md` §3 (streaming deferred to v0.2), §4 `max_payload_size_bytes`; no truncation/compression fields found in schemas

**Notes:** Hook bodies (e.g. tool results, messages, skill definitions) can be very large. v0.1 does not support streaming ACS hook traffic (explicitly deferred to v0.2). There is also no defined truncation mode or wire flag that a payload was truncated — only ClientHello `max_payload_size_bytes` ("willing to send"), with no matching Guardian limit, refuse error, or `truncated`/`content_hash`-only alternative. Compression is likewise unspecified (no content-encoding negotiation on the ACS envelope/transport beyond whatever HTTP might do out of band). Spec should say whether oversized content is refused, hashed/summarized, compressed, or streamed later, and how a Guardian knows it did not see the full body.

Even in v0.1 the Guardian should be able to declare its own receive limit (e.g. on ServerHello), not only the Observed Agent's send willingness — otherwise the Guardian cannot bound memory/DoS risk on the wire.

---

## Document authorization paradigms without forcing them into Concepts

**Where:** IBAC/FIDES/CaMeL/AARM used across Instrument and Topics; no dedicated page. Concepts README altitude rule: cross-cutting → Concepts, mechanism → pillar

**Notes:** Readers need a clear definition of the named paradigms (not common OWASP terms; lightly expanded today). But authz/policy evaluation is Instrument-mechanism — single-pillar — so a Concepts page would break the altitude rule Concepts exists to enforce. Prefer an Instrument page (or a Topics explainer that points into Instrument §6.2 / §12) for paradigm → wire mapping, engines (OPA/Cedar), and "optional citation style, not required product." Keep Intent/Capability/Trust in Concepts as the horizontal invariants paradigms consume.

---

## Stop restating structured decision fields in reasoning examples

**Where:** `docs/topics/ACS_in_action_example.md` (e.g. FIDES deny `reasoning`); also other paradigm examples there. Spec: `docs/spec/instrument/specification.md` §6.1

**Notes:** Example `reasoning` strings repeat what already lives in `reason_codes`, `policy_references`, `policy_data`, and `cited_provenance_ids` (provenance ids, paradigm check, policy id/version). §6.1 defines `reasoning` as a single *human-renderable* explanation and tells UIs/meta-policies to switch on `reason_codes` rather than parse prose. Examples should show short user-facing text (or omit fluff), not a second encoding of the structured payload — otherwise readers learn to dump structure into `reasoning`.

---

## Clarify what control paradigms mean for implementers

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite"; also Instrument §6.2 (and whatever Instrument/Topics page eventually defines paradigms — not Concepts)

**Notes:** The section presents IBAC, FIDES, CaMeL, and AARM as if they were ACS features a Guardian might be expected to ship. Unclear: must a Guardian implement all, some, or none? Are they negotiated on the wire? Are they normative ACS profiles or external research architectures ACS merely accommodates?

From the rest of the docs: the wire is paradigm-neutral (`policy_data` / `reason_codes` / `policy_references`); handshake negotiates ACS profiles (e.g. `acs-provenance` for IFC-style paradigms), not paradigm names; pure IBAC needs no Provenance profile. There is **no formal declaration** of "the enforcement paradigm" anywhere — not in ClientHello/ServerHello, not as an enum on `sessionStart` (`policy_mode` is `scope_mode`, not IBAC/FIDES/…). So statements like "Intent is optional but normative when IBAC is the enforcement paradigm" hang a MUST on an undeclared deployment choice. That "optional policy style, cite what fired" story should lead the paradigms section — with links to the source papers/specs and an honest note on how complete/implementable each mapping is — rather than four peer denial examples that read like a product matrix. Either declare paradigm(s) on the wire / in a profile, or rephrase those MUSTs as "when the deployment uses Intent/IBAC-style policy."

---

## Show conventional authz in ACS-in-action, not only research paradigms

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite" (IBAC, FIDES, CaMeL, AARM); contrast Instrument §12.1 (OPA/Rego as v0.1 reference, Cedar as v0.2)

**Notes:** The worked example leads with research control paradigms and never shows a conventional authorization policy (allow/deny on identity, role, resource, or a plain OPA/Cedar rule). Readers can infer that ACS is for FIDES/CaMeL/AARM-style systems only. Add at least one ordinary authz example (and preferably lead with it), since the deterministic layer's starting reference is OPA/Rego and paradigms are optional citation styles, not the product.

---

## Clarify whether Approver is optional and who resolves ASK

**Where:** `docs/concepts/agents.md` (Approver); Instrument §9 / §9.2; ASK as Core MUST-support disposition

**Notes:** Unclear whether an Approver is always required when ASK is used, or optional. Two models are mixed: (1) Concepts/§9 — the Guardian consults a third-party Approver (`ask_details.approver` endpoint) while the Observed Agent pauses; (2) §9.2 — some Observed Agents "route" ASK / need "approver UX", which implies ASK is returned to the main agent for local resolution (e.g. IDE user). Spell out whether the Guardian MAY return `ask` to the Observed Agent without a designated Approver, who presents the approval UI, and when Approver may be omitted entirely (e.g. never raise ASK; or substitute DEFER/DENY only).

---

## Rewrite the opaque "trust basis, not that it arrived" sentence

**Where:** `docs/concepts/agents.md` — Trust between agents: "How much weight a given fact carries depends on its trust basis, not on the fact that it arrived."

**Notes:** The last sentence is opaque without already knowing [Trust basis](docs/concepts/trust.md). Say plainly that arrival on the wire does not make a claim true — reliance follows how the fact was produced (asserted vs framework-attached vs cryptographically attested), and point at Trust basis. Do not rely on "weight" / "trust basis" jargon in the punchline.

---

## Clarify Guardian identity vs the policy (and policy-author)

**Where:** `docs/concepts/identity.md` — "Guardian identity: which policy authority is deciding."; also `docs/topics/core_concepts.md`

**Notes:** Wording makes Guardian identity sound like "the policy." Elsewhere ACS keeps three distinct: Observed Agent, Guardian, and **policy-author** (conformance: policy-author ≠ Guardian). Spell out that Guardian identity is the deciding *runtime/party* (its own principal), not the policy document and not the policy author — and say what (if anything) carries that identity on the wire in v0.1.

---

## Fix broken lead sentence on Identity overview

**Where:** `docs/identity/overview.md` — "There are 5 unique identity challenges that this standard aims to solve, where existing use cases that traditional identity models cannot support."

**Notes:** Editorial: the clause after the comma is ungrammatical (dangling "where … that …"). Rewrite into clear prose — e.g. five challenges that traditional identity models cannot support for agent use cases — and drop filler ("unique", "aims to solve") per STYLE.md.

---

## Drop the false "build-time vs execute-time supply chain" claim

**Where:** `docs/identity/overview.md` — "The supply chain you signed off on at build time is not the supply chain that executes."

**Notes:** Wishy-washy and misleading. Tools, runtime environment, and often prompts are stable; this is not primarily a supply-chain problem. The real point is non-deterministic *control flow* / action composition at runtime (which tool, order, arguments). Rewrite without the supply-chain metaphor.

---

## Do not imply agentic identity stops the "valid token, bad outcome" failure class

**Where:** `docs/identity/overview.md` — "Agent systems are introducing a new class of failures where every token is valid, every API call is authorized, and the outcome is still a security incident." (and the Unit 42 example that follows)

**Notes:** The failure class is real, but agentic identity does not solve it — it enables better governance and forensics (who acted, under which chain/intent). Prompt injection and similar attacks still succeed against valid credentials. Say that clearly so readers do not think naming/chaining identity closes the incident class.

Also, these two sentences do not follow: "The failure occurred because the runtime could not distinguish user intent from adversarial instructions introduced during execution. This illustrates the central identity challenge of agent systems." Failure to separate user intent from injected instructions is a runtime-control / intent-enforcement problem, not (by itself) an identity challenge. Either retarget the punchline to Instrument/Intent, or explain the actual identity-shaped gap the example is meant to show.

---

## Restructure the five identity-challenges table for readability

**Where:** `docs/identity/overview.md` — "The 5 Unique Runtime Identity Challenges for Agents" table

**Notes:** Editorial: cells are long multi-paragraph "poems" (threat narrative, standards gap, desired outcome, status crammed into one row). Hard to scan in MkDocs. Prefer one short row per challenge with a link into a dedicated subsection (or definition list / cards), keeping the comparison surface tight.

---

## Stop calling the identity challenges "unique"

**Where:** `docs/identity/overview.md` — "5 Unique Runtime Identity Challenges" (heading and body); at least Over-Privilege and Token Theft Resistance are long-standing IAM problems

**Notes:** Drop the uniqueness claim. The challenges are real; they are not unique to agents.

---

## Separate chain-integrity forensics from chain-based policy enforcement

**Where:** `docs/identity/overview.md` — Chain Integrity challenge (and related identity-standards prose)

**Notes:** Naming and verifying a delegation chain is clearly useful for forensics. It is much less clear that real-life policy enforcement can be driven by long, complicated chains. Human-authored policies cannot reasonably express rules over deep chains, and there is little evidence of other authz systems that make effective runtime use of them. Split the ask: audit/reconstructability vs enforcement, and do not treat "verify the full chain at every hop" as an obvious policy input without showing how Guardians would consume it.

---

## Correct the false claim that API keys are always bearer secrets

**Where:** `docs/identity/overview.md` — Token Theft Resistance row: "API keys are bearer secrets by definition."

**Notes:** False as a universal claim. Many "API keys" / access-key pairs are used to MAC or sign requests (e.g. AWS SigV4), not presented as bearer credentials. Say that *bearer* API keys / opaque tokens are possession-sufficient; do not flatten all API-key schemes into bearer.

---

## Do not call GNAP and AMR "broadly adopted"

**Where:** `docs/identity/standards.md` — Existing Standards intro: "The following standards are broadly adopted…"

**Notes:** The blanket claim is false for at least **GNAP (RFC 9635)** (published, little real-world deployment) and **AMR (RFC 8176)** (OIDC auth-method claim; not broadly adopted as a foundation of modern identity). Soften or split the intro (e.g. "established / emerging") and do not put GNAP and AMR in the same adoption bucket as OAuth 2.0 / OIDC.

---

## Do not claim token issuance cannot support per-action authz

**Where:** `docs/identity/standards.md` — Why OAuth Alone Doesn't Meet Agent Needs table: "No token-issuance standard can patch this; it requires inline behavioral verification at action time."

**Notes:** Overstated. OAuth can carry fine-grained authorization (e.g. **RAR**, which this page already says ACS requires) so token issuance does not contradict per-action authz enforcement. The real gap vs a hijacked client is detecting misbehavior / verifying intent at runtime — not that issuance standards cannot express action-scoped policy. Narrow the claim accordingly. Same table elsewhere lists OAuth extensions as mitigations; this row alone reads as if OAuth can never be a workable solution — inconsistent framing.

---

## Short TTL alone is not enough for high-risk actions

**Where:** `docs/identity/standards.md` — refresh-token mitigation row and Token lifetimes summary: proposed "60 s for high-risk actions"

**Notes:** A 60-second ceiling still leaves a wide abuse window; plenty of damage fits in a minute. Time-limited tokens are not the solution for high-risk actions. Call for (or at least acknowledge) **single-use** / one-shot credentials where the risk warrants it, rather than treating aggressive TTL as sufficient.

---

## Do not treat fail-open as an open peer option to OAuth/AIMS

**Where:** `docs/identity/standards.md` — IdP reachability row: "Neither OAuth nor AIMS specifies a failure-mode contract." → open work item "fail-closed vs. fail-cached vs. tier-dependent policy."

**Notes:** The absence of an explicit failure-mode clause is not a hole OAuth/AIMS left for fail-open: the community reads them as **fail-closed** (deny when the IdP cannot authorize). Framing an ACS open item as choosing among fail-closed / fail-cached / tier-dependent as peers understates how risky fail-open is here. Fail-open can make sense when the PDP is outside the attacker's reach, so unavailability is mostly non-malicious (ops failure). That does **not** hold for the Guardian: the Observed Agent already has a wide attack surface against it (prompt injection into the Guardian, and more), starting with simple **DoS** that induces decision failure and then rides fail-open. Prefer fail-closed as the default posture; treat any fail-open or fail-cached path as an exceptional, tightly justified deviation.

Related (not a full duplicate): [#32](https://github.com/GenAI-Security-Project/agent-control-standard/issues/32) (refusals should fail closed), [#37](https://github.com/GenAI-Security-Project/agent-control-standard/issues/37) (fail-open audit event shape).

---

## Immutable Intent needs a real integrity story, not only an invariant

**Where:** `docs/concepts/intent.md`; `docs/spec/instrument/specification.md` §8.4; identity pages that cite "immutable intent"

**Notes:** The spec states a normative immutability rule (`Intent.parsed` MUST NOT be widened by the LLM, tool outputs, or untrusted channels; only via ASK/`intent_extension`) and says the framework MUST ignore/reject attempts. That is an *invariant*, not a *mechanism*. It does not recommend how Intent is sealed at establishment (who parses `raw` → `parsed`, how `parser_provenance.origin: user_input` is verified rather than self-asserted), how a host/Guardian prevents the Observed Agent from supplying or rewriting Intent before commit, or how to stop adversarial reframing that never mutates the capability list. Related but narrower: [#149](https://github.com/GenAI-Security-Project/agent-control-standard/issues/149) (canonical Intent when `sessionStart` carries none).

---

## Intent is on the wire; stop calling it "not a wire format"

**Where:** `docs/concepts/intent.md` opener ("Intent is a governance concept, not a wire format"); hook schemas that embed Intent (`session-start.json`, `agent-trigger.json`, `subagent-start.json`); §8.4

**Notes:** Contradictory. Intent *is* carried on the wire (`raw`, `parsed`, `parser_provenance`, `scope_mode`), so the opener undercuts the schemas. Either extract a shared Intent schema and document the on-wire types in prose (at minimum: `raw` is a free-text string; `parsed` is a capability list — today only the JSON Schema says `raw: string`, and the concept page never does), or drop the "not a wire format" line and say Intent is a governance concept *with* a wire representation.

---

## Document goal→capability derivation for Intent.parsed

**Where:** `docs/concepts/intent.md` (`Intent.parsed` = "the set of things the agent is permitted to do"); also `docs/topics/ACS_in_action_example.md` (example Intent `["summarize Project X"]` vs schema capability tuples)

**Notes:** In real chat, humans state what the agent is *required* to do ("summarize this doc"), not an exhaustive permission set ("you may read this doc, related docs, write a summary, fetch from the Internet for reading…"). Those permissions are **derived**. ACS never says who performs that derivation (host, Guardian, separate parser), what inputs it uses, how over-/under-approximation is handled, or how the derivation is audited beyond `parser_provenance`. The example even stores a goal string as Intent while the wire schema expects `{tool, operation, resource}` tuples — reinforcing the gap. Either specify the derivation contract or stop presenting `Intent.parsed` as if users author the permission set directly.

---

## Define a capability registry (or admit policy is deployment-local)

**Where:** `docs/concepts/capability.md`; §11 ("Capability vocabulary uses abstract names…"); AgBOM / `tool-call-request.json` (`capability` as free string with examples)

**Notes:** ACS does **not** define a concrete capability set (e.g. `filesystem.delete`) and provides **no registry** — only illustrative dotted names. Without a shared vocabulary (IANA-style registry, profile catalogs, or at least a core set + extension rules), Guardians, Intent.parsed, and tool declarations cannot interoperate across vendors: the same action can be labeled differently and policy bundles are not portable. Either ship a registry (core identifiers + registration/extension process) or explicitly scope IBAC interoperability as deployment-local string matching. Also note shape drift: concept/examples use capability IDs, while Intent.parsed schema uses `{tool, operation, resource}` tuples. This is the *permission* vocabulary, not the tool-call action syntax — see the separate note on a core tool-action subset.

---

## Untangle Capability vs agent_capability in the concepts glossary

**Where:** `docs/concepts/skill.md` ("A skill differs from an `agent_capability`…"); `docs/concepts/capability.md`; concepts README

**Notes:** In the section that is supposed to define terms, "Capability" means at least two things: (1) an abstract permission ID (`filesystem.delete`) on the Capability page, and (2) the AgBOM `agent_capability` component (a passive grouping of tools/MCP/peers). Skill.md contrasts skill with `agent_capability` but links that term to `capability.md`, which mostly defines (1). Readers cannot tell whether "Capability" in the eight-concept list is the permission vocabulary, the inventory grouping, or both. Rename or split (e.g. keep Capability for abstract permissions; call the AgBOM type something that does not collide) and fix the cross-links so the glossary does not overload the word.

---

## Rewrite "trust launder" jargon in Provenance/Trust openers

**Where:** `docs/concepts/provenance.md` opener ("lets trust launder silently"); also `docs/concepts/trust.md` ("trust laundering up the spectrum")

**Notes:** Opaque metaphor for a first-line definition. Say what happens in plain terms: if provenance/trust basis is dropped at a boundary, untrusted or merely asserted data can be treated as trusted/attested on the far side. Keep "laundering" only if glossed once; the Provenance opener should not depend on readers already knowing the Trust page phrase.

---

## Separate channel/metadata integrity from content trust on the Trust Basis spectrum

**Where:** `docs/concepts/trust.md` — spectrum ("Each rung is a stronger ground for reliance")

**Notes:** The ordered spectrum invites conflating two different questions. Framework-deterministic attachment of provenance (e.g. "we read this from the DB") does **not** raise trust in the text if that text arrived from an untrusted source — it only strengthens confidence in the *origin label*, not the content. Cryptographic signing/hashing similarly authenticates emitter/integrity of bytes; signing untrusted content adds nothing to how much the *substance* should be believed. Rewrite so "Deterministically attached" and "Cryptographically attested" are clearly about reliability of ACS metadata/channel claims (provenance fields, envelopes, audit chain), not a ladder that elevates trustworthiness of the underlying data. The "rungs do not collapse" invariant does not fix this if the rungs are already mis-sold as content-trust levels.

---

## Clarify that profile "layers" are declared, not proven

**Where:** `docs/spec/conformance.md` — diagram intro: "Each layer is a self-declared claim from the handshake, not a verified fact."

**Notes:** Cryptic wording, not a deep protocol flaw. A reader of the stack diagram hears "layer = claim from handshake," which does not parse. Intended meaning (said more clearly later): advertising `profiles_supported` / `profiles_accepted` is self-assertion. That pattern is normal for protocol capability negotiation — strong verification of peer capabilities is rare; systems usually rely on implicit trust assumptions. Rewrite the caption in plain terms (optional conformance profiles; handshake names are advertisements) without sounding as if unverified capability ads are uniquely broken. Keep the "Who verifies…" paragraph for procurement/conformance honesty; do not dramatize ordinary handshake self-declaration on the figure.

---

## Call the wire a JSON-RPC 2.0 profile — there are no real JSON-RPC extensions

**Where:** `docs/spec/conformance.md` / §3 ("JSON-RPC 2.0 with ACS extensions"); `request-envelope.json` / `response-envelope.json` descriptions

**Notes:** As schematized, ACS does **not** extend JSON-RPC 2.0 at the protocol layer: top-level members are only the standard ones (`jsonrpc`, `method`, `id`, `params` / `result`|`error`). ACS fields (`acs_version`, `signature`, decisions, …) sit inside `params`/`result`, which is ordinary application payload. What ACS adds is a **constrained profile** (no notifications, method namespaces, required ACS `params` shape, `additionalProperties: false`, optional batching). Prefer that wording over "with ACS extensions." The one place prose *claims* a non-conforming extension — §3 "top-level `acs_version`" — contradicts the schema; that is already [#194](https://github.com/GenAI-Security-Project/agent-control-standard/issues/194).

---

## Justify hook negotiation / out-of-band capability vs normative emit-when-applicable

**Where:** `docs/spec/conformance.md` (Guardian determines subagent-emission capability out-of-band); §4 `methods_implemented` / `methods_evaluated`; similar ASK/MODIFY "deployment-defined means" precedents

**Notes:** Why must the Guardian know which hooks are "active"? Alternative: make emission normative on the harness (*if the framework can observe/spawn X, it MUST emit the corresponding hook*) and treat absence as "event did not occur," without out-of-band capability tables or negotiated hook subsets. Today the Guardian is asked to infer client ability via identity/policy configuration — fragile and duplicates what a MUST-when-applicable rule would already require. Handshake method lists and `methods_evaluated` need an explicit rationale (partial Guardians? optional observability hooks?) or should be dropped in favor of a fixed Core hook set plus normative conditional emission. Related (coverage detection, not the design ask): [#16](https://github.com/GenAI-Security-Project/agent-control-standard/issues/16), [#31](https://github.com/GenAI-Security-Project/agent-control-standard/issues/31).

---

## Derive signature algorithm from the key, do not trust wire `algorithm`

**Where:** §10 signature envelope `{ algorithm, value, key_id }`; hooks.md common-envelope example; `request-envelope.json` `Signature`

**Notes:** Best practice is to bind algorithm to the key material looked up via `key_id` (and reject mismatches), not to take `algorithm` from the message as authoritative. Explicit wire `algorithm` invites algorithm-confusion / downgrade games (claim a weaker or wrong alg for the same key). Prefer `key_id` + `value` only, with alg (and hybrid composition) carried in the key descriptor; if `algorithm` stays for debugging, it MUST be ignored for verification except as a must-match check against the key. Hybrids already say `key_id` resolves to a descriptor that pins both keys — that is the right place for alg identity.

---

## Say what agentTrigger MODIFY rewrites — not vague "before activation"

**Where:** `docs/spec/instrument/hooks.md` — agentTrigger: "Guardian MAY rewrite the trigger payload (e.g. redact PII) before activation."

**Notes:** "Activation" is opaque here. If MODIFY is meant to change what the agent *consumes* when starting on the trigger, say that in the same plain terms as `userMessage` ("before delivery to the agent"). If the rewrite is only for the audit record, say "audit" and do not imply the live trigger path. As written, the sentence confuses the gate and the audit trail.

---

## Make modified_content a structured payload replacement, not a string

**Where:** `specification/v0.1.0/modifications.json` (`modified_content: string`); §6.3 ("opaque replacement string")

**Notes:** Wholesale MODIFY is typed as a string, but hook payloads are structured objects (e.g. `userMessage.content` is an array of typed parts; tool calls are objects). A string cannot replace those without an unspecified encoding (JSON-as-string?). Type `modified_content` as the replacement **payload object** (same shape as the hook's `params.payload`, or at least `object` / open JSON value), and keep `redactions` / `parameter_overrides` for partial edits. Related host pain: [#181](https://github.com/GenAI-Security-Project/agent-control-standard/issues/181).

---

## Specify a core subset of tool-call actions, extensible later

**Where:** `specification/v0.1.0/hooks/tool-call-request.json` (`tool.name`, optional `operation`, open `arguments`, optional `raw_command`); not the capability vocabulary

**Notes:** Tool calls are not capabilities. A capability is an abstract permission label; a tool call is the concrete action (what ran, with which arguments). ACS defines neither a syntax nor a catalog for the latter: `name` / `operation` / `arguments` / `raw_command` are free-form, so there is no shared shape for the common cases (shell/syscall, file read/write/delete, network connect/fetch). Put a small normative subset of those actions in the main spec — typed arguments, not just a string id — and define an extension point for everything else. Do not wait for a full tool ontology, and do not treat dotted capability examples as that subset.

---

## preCompact DENY is not a durable policy once the window is full

**Where:** `docs/spec/instrument/hooks.md` — preCompact: Guardian MAY DENY compaction (e.g. until trusted re-grounding)

**Notes:** The hook is realistic as a lineage chokepoint (`postCompact` binding `derived_from` to the union of compacted entries). DENY is not. `triggered_by: size_threshold` means the runtime is already out of room; refusing compaction does not keep the uncompacted context usable. A "don't compact untrusted data" policy has to say what happens instead (drop specific entries, compact with tainted lineage preserved, end the session) — not an open-ended block. Treat size-forced compaction as inevitable and put the control on what the summary is allowed to claim, not on whether compaction occurs.

---

## Do not present "fabricate a chain hash" as an option

**Where:** §8.6; `docs/spec/instrument/hooks.md` subagentStop; `subagent-stop.json` — "omits the field rather than fabricate a value"

**Notes:** Omitting `final_chain_hash` when there is no session chain is fine. Naming fabrication as the alternative is not: a standard should not contemplate a forged hash as a choice the implementer is steered away from. A made-up value would also fail immediately against the published chain head (`CHAIN_MISMATCH`), so it is not a workable shortcut. Say the field is absent when the framework has no chain, and stop there.

---

## Do not push skill vetting entirely outside ACS

**Where:** `docs/spec/instrument/hooks.md` skillRegister ("Marketplace pre-publication scanning and publisher reputation are out of scope"); `specification/proposals/skill-lifecycle/README.md` § Scope boundary ("the marketplace's job")

**Notes:** The stated split is that ACS only offers the register/load control point, and scanning plus publisher reputation belong to the marketplace. That drops two things enterprises need. Prior scan results and a publisher/registry decision are how you avoid re-analyzing the skill body on every register or load (performance), and how an enterprise states its own vetting policy ("only digests in our registry," "this publisher," "this scan passed") instead of re-deriving trust from the raw definition each time. ACS does not have to run the scanner. It does need a place to carry a prior attestation, publisher identity, or enterprise-registry decision into `skillRegister` / `skillLoad` so policy can rely on it. `registration_provenance` does not fill that place: it is a normal Provenance object (`origin`, optional `source_id`, `derived_from`). Prose says `source_id` SHOULD name a publisher and a Guardian MAY require a signed provenance, but the schema has no signature, scan result, or registry verdict.

---

## Do not file Inspect methods under Instrument "Supported Hooks"

**Where:** `mkdocs.yml` — Specification → Instrument → Supported Hooks (`docs/spec/instrument/hooks.md`); the same page's `agbom/snapshot` and `agbom/changed` rows

**Notes:** The nav presents one hook catalog as Instrument. The page itself says `agbom/*` are Inspect-pillar methods, and § Instrument says they are not Instrument hooks — they only share the request envelope and the audit chain. `system/ping` is the same kind of misfit (liveness, not a control hook). Keep a shared envelope reference if useful, but list Inspect methods under Inspect (they are already summarized in `docs/spec/inspect/README.md`) so "Supported Hooks" is not a grab-bag of every method that happens to be decision-eligible.

---

## (Title of next comment)

**Where:**

**Notes:**
