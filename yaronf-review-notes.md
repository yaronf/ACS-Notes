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

**Where:** `docs/concepts/identity.md` (and related identity type examples); `docs/spec/instrument/specification.md` §11 (identity descriptor `type`: `posix_uid`, `windows_sid`, `oauth_subject`, `cert_subject`, …); Identity workstream descriptor schema

**Notes:** SECEVENT published [RFC 9493](https://www.rfc-editor.org/rfc/rfc9493.html) (Subject Identifiers for Security Event Tokens): typed JSON subjects with a `format` discriminator (`iss_sub`, `email`, `uri`, `did`, `opaque`, …) and an IANA registry. `iss_sub` is a better stand-in for OAuth/OIDC principals than ACS's illustrative `oauth_subject` + bare `sub` (issuer+subject is what makes the id unique). Adopted mainly via OpenID Shared Signals / CAEP / RISC (e.g. Okta SSF); not a universal OAuth wire replacement yet. Worth aligning ACS identity descriptors with that registry (and using `uri`/`did` for SPIFFE/WIMSE) rather than growing a parallel type list. §11 is that list in the specification: a `type` discriminator with `posix_uid`, `windows_sid`, `oauth_subject`, `cert_subject`, and an ellipsis, with the schema deferred to the Identity workstream. The same alignment applies there.

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

**Notes:** Best practice is to bind algorithm to the key material looked up via `key_id` (and reject mismatches), not to take `algorithm` from the message as authoritative. Explicit wire `algorithm` invites algorithm-confusion / downgrade games (claim a weaker or wrong alg for the same key). Prefer `key_id` + `value` only, with alg (and hybrid composition) carried in the key descriptor; if `algorithm` stays for debugging, it MUST be ignored for verification except as a must-match check against the key. Hybrids already say `key_id` resolves to a descriptor that pins both keys — that is the right place for alg identity. Doing so does not damage the crypto agility §10.1 claims. That claim is the registry plus the handshake-negotiated set, and the migration sentence already says only that set shifts, with no wire-format change. A new algorithm is a new key descriptor and a new handshake entry; the per-message `algorithm` string is not what makes the envelope agile.

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

## Say the LLM must not emit hooks, not "the agent"

**Where:** `docs/spec/instrument/specification.md` §1 principle 2: "The agent MUST NOT have knowledge of hooks."

**Notes:** The sentence fights the rest of the spec. §2 defines the Observed Agent as the party that implements ACS endpoints and sends hook traffic, and §6.4 requires it to wait for and apply ALLOW, DENY, MODIFY, ASK, and DEFER. Something in that runtime has to know the hooks exist, or an ASK or MODIFY response has nowhere to land. The following sentence is the real rule: provenance is populated outside the LLM's output path. Rewrite the principle as "the model MUST NOT emit or observe hook traffic; the framework does." Do not use "agent" for both the harness and the model in the same principle list.

---

## Move the Instrument "design principles" to ACS-wide scope

**Where:** `docs/spec/instrument/specification.md` §1 Design Principles

**Notes:** The section sits under the Instrument specification, but the four principles are not about interception and enforcement. (1) is the whole-spec contract-versus-implementation split. (2) and (3) are provenance and trust classification, which the same page treats as a cross-cutting profile. (4) is explicitly about all three pillars and tiered conformance. A reader of Instrument does not get principles for hooks, dispositions, or the Guardian decision path. Put ACS-wide principles with the spec introduction, and if Instrument keeps a principles list, make it about the control loop.

---

## The Observed Agent calls the Guardian; it does not implement ACS endpoints

**Where:** `docs/spec/instrument/specification.md` §2: "Observed Agent … Implements ACS endpoints (or the stdio analog) and sends hook traffic to the Guardian."

**Notes:** The same section's flow has the Observed Agent HTTP POST or stdio-write the hook and wait for the response. Handshake names are ClientHello from the Observed Agent and ServerHello from the Guardian. The Guardian is the party that accepts the call. "Implements ACS endpoints" describes the server, and it contradicts the second half of the same sentence. Say the Observed Agent emits hook requests and applies the decision; the Guardian implements the ACS endpoint.

---

## Specify the Guardian's two-layer handoff, or stop describing one

**Where:** `docs/spec/instrument/specification.md` §2 (chain config `*`, `on_ask`, pattern-based) and §12.1–12.2 (`delegate_to: "agent"`, intermediate output)

**Notes:** The interaction is named twice and specified neither time, and the two accounts do not match. §2 says a chain config decides when the LLM layer runs. §12.1 says the deterministic engine returns an optional `delegate_to: "agent"`. There is no schema for the chain config, for `delegate_to`, or for the "intermediate output" the agent layer receives. Nothing says whether the agent layer's decision replaces the deterministic one, whether the deterministic layer can veto it, or what happens on agent-layer timeout besides "timeouts follow `timeout_config`." The agent layer is OPTIONAL, so a deterministic-only Guardian needs no handoff. A deployment that uses both needs the handoff written down.

---

## Replace the methods_implemented prefix pattern with a real method-name rule

**Where:** [`handshake.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/handshake.json) `ClientHello.methods_implemented` (`^(steps/|protocols/|agbom/|trace/|system/|handshake/|wrapped:).+`); the same pattern on `request-envelope.json` `method`; `ServerHello.methods_evaluated` has no pattern

**Notes:** The pattern is valid ECMA-262 and every documented method (`steps/toolCallRequest`, `handshake/hello`, `agbom/snapshot`, `protocols/MCP/tools/call`, `wrapped:mcp-2025-06-18/tools/call`, …) matches it. It does not constrain method names. `.+` is any non-empty tail, so `steps/*`, `steps/notAHook`, `steps/toolCallRequest extra`, `trace/foo` (v0.1 defines no `trace/*` methods), and `wrapped:!!!` all validate. JSON Schema `pattern` is an unanchored search: this expression has `^` and no `$`, and the version patterns in the same file are end-anchored. In JavaScript, `steps/toolCallRequest\nignored` matches because the search succeeds on the first line. `methods_implemented` may also be empty or contain duplicates (`minItems` and `uniqueItems` are absent). `methods_evaluated` items are bare strings, so a ServerHello can name a method the ClientHello pattern would reject; `timeout_config.per_method_ms` keys are the same. The description says this list is how the Guardian knows which hooks to expect and which it can treat as not-emitted. The pattern cannot support that. Use one method-name grammar (known names, or a tight `namespace/Name` form plus the `wrapped:` form), end-anchor it, and apply it to `methods_implemented`, `methods_evaluated`, and the request `method`. Related, not the same ask: [#15](https://github.com/GenAI-Security-Project/agent-control-standard/issues/15) (how the two lists resolve), and the hook-negotiation note above.

The file also does not validate a handshake message at its root. The root schema is `type: object` with no properties, so an empty object validates against the published URL, and `acs_schema.json`'s `$ref` to `handshake.json` (no fragment) does too. Only `#/$defs/ClientHello` and `#/$defs/ServerHello` are the message schemas. The request envelope's `payload` is an unconstrained object, so `handshake/hello` is not checked against `ClientHello`.

---

## Drop transport negotiation from the handshake, or define a switch it can actually perform

**Where:** [`handshake.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/handshake.json) `ClientHello.transports_supported` (array of `http` | `https` | `stdio`) and `ServerHello.selected_transport` (one of those strings); `docs/spec/instrument/specification.md` §4 and §13 (`system/ping` "MAY use it to renegotiate transport")

**Notes:** `selected_transport` is one value, which is the only shape a selection can have. The plural sits on the other side: `transports_supported` is an array, and the §4 diagram says the client offers "transports." The schema has no description on `selected_transport` itself. The enum lists the three legal names; it is not a multi-value result.

The negotiation still does not do anything. `handshake/hello` is already a request on a concrete transport: an HTTP POST or a stdio write. TLS, or its absence, was settled when that connection opened. A stdio channel and an HTTP endpoint are different process topologies, and the ServerHello carries no address, port, or command for the one it names. Nothing says what the Observed Agent does when `selected_transport` differs from the transport the hello arrived on, or that the value must be a member of `transports_supported`. §13 says a failed `system/ping` MAY cause the agent to "renegotiate transport," and then defers `system/handshake_renegotiate` to v0.2. The reference Guardian writes `selected_transport: "http"` without reading the ClientHello, and the reference host never reads the field back. Either delete both fields in v0.1, or specify the switch: which endpoint, when subsequent hooks move, and what a mismatch with the current connection means.

---

## Define the ServerHello fields the §4 list only names

**Where:** `docs/spec/instrument/specification.md` §4, the ServerHello field list; [`handshake.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/handshake.json) `$defs/ServerHello`

**Notes:** That sentence is an inventory, not a set of forward references. Later prose defines `on_decision_failure` (§4.1, §6.4), `skew_window_ms` (§10.3), `profiles_accepted` (Conformance), and, thinly, `timeout_config` (the negotiated timeout in §6.4 and §12; the `default_ms` / `per_method_ms` shape is schema-only), `policy_requires_provenance` (the next paragraph of §4), `agbom_serializations_supported` (one sentence in the Inspect README), and `trace_emission` (one sentence on the OTel page, about the collector endpoint only). `negotiated_version` is the version-mismatch sentence plus the schema. `signature_algorithms_supported` is not named again; §10.1 says the handshake declares algorithms and then defines the registry, while the schema says an empty array means signatures are not required. `methods_evaluated` and `approver_types_supported` appear in the docs only in that list; their behavior is a schema description, and §9.2 says ASK capability is not on the wire. `selected_transport` has no description at all. The schema's `trace_emission` descriptions cite "Section 3.15", which this specification does not contain. Give each field a normative sentence, or stop listing it. Related, not the same ask: [#15](https://github.com/GenAI-Security-Project/agent-control-standard/issues/15) (how profiles, methods, and signature algorithms resolve).

---

## A Guardian cannot attach to a session; the Observed Agent is the only caller

**Where:** `docs/spec/instrument/specification.md` §4.1: "Whether a Guardian can attach mid-flight to a session that started unguarded is undefined in v0.1"

**Notes:** The sentence gives the Guardian an action the protocol does not have. Every ACS message in v0.1 is a request the Observed Agent sends: ClientHello, hooks, `system/ping`. The Guardian answers. There is no Guardian-initiated call, and v0.1 has no notifications, so a Guardian that was down when the session started has nothing to send into that session. The same paragraph already has the Observed Agent as the actor ("SHOULD retry the handshake for subsequent sessions"). The deferred `system/handshake_renegotiate` is also a `system/*` method, which the Observed Agent would send. Say whether the Observed Agent may begin a handshake for a `session_id` it already started unguarded, and what happens to the actions it took before that hello. Do not leave "the Guardian attaches" as an undefined capability.

---

## Make the §5 hook table and the Hooks page say the same thing about decisions

**Where:** `docs/spec/instrument/specification.md` §5 Hook Taxonomy; `docs/spec/instrument/hooks.md` Overview table and each hook's **Decision** line

**Notes:** The 19 `steps/*` names and their order match. The decision story does not. §5 writes "decision-eligible" on only three rows (`preCompact`, `skillRegister`, `skillLoad`) and says nothing about the other sixteen. The Hooks overview has a Decision-eligible column: Yes for those three and for `sessionStart`, `agentTrigger`, `turnStart`, `userMessage`, `agentResponse`, `knowledgeRetrieval`, `memoryContextRetrieval`, `memoryStore`, `toolCallRequest`, `toolCallResult`, and `subagentStart`; No for `postCompact`, `subagentStop`, `skillUnload`, `turnEnd`, and `sessionEnd`. A reader of §5 cannot tell an unmarked hook from an audit-only one. The Hooks page then disagrees with itself on `postCompact`: the overview says not decision-eligible, and the hook body says the Guardian MAY return MODIFY and MAY NOT return DENY. One catalog should state the legal dispositions per hook. Related, not the same ask: [#146](https://github.com/GenAI-Security-Project/agent-control-standard/issues/146) (the phrase "not decision-eligible" means two things, and `subagentStop` disagrees between schema and `hooks.md`), [#148](https://github.com/GenAI-Security-Project/agent-control-standard/issues/148) (schemas leave eligibility blank for most hooks). §5 is a third surface neither issue cites.

---

## Tag natural-language inputs and displayed text with a language

**Where:** `steps/userMessage` and `steps/agentResponse` content parts (`value` with `type: text`); `Intent.raw` on `sessionStart` / `agentTrigger`; `toolCallRequest` natural-language framing; retrieval, memory, and compaction summary text; response `reasoning` ([`response-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/response-envelope.json)); ASK `question`, `context`, and `options` ([`ask-details.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/ask-details.json))

**Notes:** These fields are bare strings. No schema carries a language tag. That matters in two directions. On the way in, several of the strings are what a policy or an intent parser decides on: user text, `Intent.raw`, a tool-call rationale, retrieved or remembered text. An untagged string is read as whichever language the rule was written in. A faux ami then either trips a deny-word that is ordinary speech in the speaker's language, or hides the same instruction because the keyword list is in another language. On the way out, `reasoning` is specified as the human-renderable explanation for end-user display, and an ASK's `question` and `context` are what an approver reads. A display that does not know the language cannot choose the right rendering, and an approver can misread the question the same way. Use a BCP 47 tag (RFC 5646) on those natural-language values, per content part where a message can mix languages, and the same tag on `reasoning` and the ASK prompt. Identifiers, enums, and hashes do not need one.

---

## Let a DENY omit reasoning when the policy must stay confidential

**Where:** `docs/spec/instrument/specification.md` §6, DENY required fields; [`response-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/response-envelope.json) (`reasoning` required when `decision` is `deny`, and also when it is `modify`, `ask`, or `defer`)

**Notes:** The prose and the schema both require `reasoning` on DENY, and the schema description says the operator or end user needs that explanation. The same response is what the Observed Agent receives. A deployment that treats the policy as confidential does not want a human-readable account of why the action was blocked delivered to the party being controlled: the text is a map of the boundary, and an attacker who can induce actions can read it back. `reason_codes` and `policy_references` are optional, so those can be left off; `reasoning` cannot. Omitting the field is invalid. A conformant DENY needs a way to carry no explanation, with the audit record kept on a channel the agent does not see. The same pressure applies to MODIFY, ASK, and DEFER, which require `reasoning` too.

---

## Say what the Observed Agent does with a DEFER

**Where:** `docs/spec/instrument/specification.md` §6 (DEFER "pauses the step until it resolves") and §9.2; [`defer-details.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/defer-details.json)

**Notes:** ASK names an approver and a question. DEFER tells the Observed Agent to pause and gives it `reason`, `resolution_method` (`additional_context`, `human_approval`, `timeout`), `resolution_timeout_ms`, `timeout_decision`, and optional `required_context`. The JSON-RPC response is already finished, and v0.1 has no notification the Guardian can send later, so nothing in that payload is a next step. `required_context` names inputs and does not say where they go. `resolution_method: human_approval` is an ASK with no `ask_details`: no approver and no question. `timeout_decision: ask` tells the agent to ask and includes neither. §9.2 uses DEFER as the substitute when the client cannot ASK, and says the block might clear through retry, an out-of-band escalation, or a later state change, still without a step. Cascading deferrals "MUST be bounded" per session, and no bound is given. Related, to link: [#14](https://github.com/GenAI-Security-Project/agent-control-standard/issues/14) (no resolution method, no correlation id, block vs retry vs give up, and `timeout_decision: ask` for a client that cannot ASK).

---

## Move the decision-failure rule out of "Honoring decisions" and finish it

**Where:** `docs/spec/instrument/specification.md` §6.4 ("A step suffers a decision failure when no usable decision arrives"); the one-off receiver rules that sit beside it (§6.3 illegal `modifications`, §6.5 a `MODIFY` the client cannot apply, §17.1 recovery column)

**Notes:** §6.4 is the attempt at one receiver rule. "No usable decision" covers silence, transport failure, a malformed response, and an error instead of a decision, and all of them take `on_decision_failure`. That is a protocol-failure rule, and it is filed under "Honoring decisions," whose job is what the Observed Agent does with a verdict that did arrive. The paragraph also stops short of the rule. It gives one posture for the step, default `proceed`, and does not give a code, a place to log, or a choice between denying the step and ending the session. "Record an audit event" does not say where: the SessionContext chain is the Guardian's, and the only deployment-local audit log in the spec is the one §4.1 uses when no Guardian answered. The neighboring sections then leave the paragraph. §6.3 fails closed on an illegal `modifications` object, which is the right outcome for that fault and a different outcome from §6.4's malformed response. §6.5 fails closed on a `MODIFY` the client cannot apply, except at `postCompact`. §17.1 gives each Guardian error a recovery sentence and never says whether the session continues. Lift §6.4 into its own protocol-failure section, and make the one-off sentences instances of it: classification including a code, where it is logged, and whether the step is denied or the session ends. Related, not the same ask: [#148](https://github.com/GenAI-Security-Project/agent-control-standard/issues/148) (what a runtime does with a disposition the hook does not allow).

---

## Say which postCompact field a MODIFY rewrites, and what the agent does with it

**Where:** `docs/spec/instrument/specification.md` §6.5, the `postCompact` exception; `docs/spec/instrument/hooks.md` `postCompact`; [`hooks/post-compact.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/hooks/post-compact.json); [`modifications.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/modifications.json)

**Notes:** Compaction has already replaced a span of context with `summary.value`. DENY cannot undo that, so the only remaining control is MODIFY, and the hook's example is "redact a region the policy can't compact." A client that cannot apply the MODIFY keeps the original summary. So the intended effect is: the Observed Agent replaces the summary now sitting in the context window. The modification object does not say that. `modified_content` is one string standing in for the whole payload, which is an object of summary, provenance, compacted entries, and chain hashes. `parameter_overrides` are tool-call arguments. A JSON-pointer redaction of `/summary/value` is the only shape that matches the example, and nothing names it. The rewritten text also has nowhere to go in provenance: `origin` MUST be `agent_generated` and `derived_from` MUST be the union of the compacted entries, both populated by the framework, and §6.3 defines no Guardian-injected origin. `post_compact_chain_hash` is already in the request, computed for the summary as submitted. Say that a `postCompact` MODIFY replaces `summary.value` in the context window, which provenance the replacement carries, and which of the two summaries the chain hash covers. Related, not the same ask: [#146](https://github.com/GenAI-Security-Project/agent-control-standard/issues/146) (the phrase "not decision-eligible" still allows this MODIFY).

---

## Schematize the SessionContext container as an extensible object

**Where:** `docs/spec/instrument/specification.md` §8, the paragraph after the SessionContext member list

**Notes:** §8 names the container (`session_id`, `chain_hash`, `entries`, `provenance_summary`, optional `intent`) and then says it is intentionally not schematized in v0.1: the children already have schemas, the container is server-side state, and cross-Guardian interoperability rides on the wire artifacts. JSON Schema already provides the mechanisms for an extensible object. Declare those members, `$ref` `ContextEntry` and `ProvenanceSummary` (and the shared Intent schema once that exists), and leave further members open with `additionalProperties` or `unevaluatedProperties`. This package already opens objects that way (`additionalProperties: true` on `platform_context`, `modifications`, and AgBOM component maps). ContextEntry, in the next subsection, is the same kind of server-side object and still has a schema: its description says storage is implementation-defined, and the schema constrains the canonical form used for hashing and audit export. That is the container's job too. The schema is what fixes `chain_hash` as the lowercase-hex head, `entries` as an array of ContextEntry, and `provenance_summary` as the existing summary, while a Guardian stays free to add fields and to store the object however it likes. The published `chain_hash` stays the wire commitment. Related, not the same ask: [#18](https://github.com/GenAI-Security-Project/agent-control-standard/issues/18) (no cross-Guardian verification protocol).

---

## Stop §10 from citing itself for the signed input

**Where:** `docs/spec/instrument/specification.md` §10 opening sentence ("A signature over the §10 canonical input"); §10.2 ("the canonical input defined in §10")

**Notes:** Editorial. Both sentences sit inside §10 and point at §10 for the bytes that get signed. The opening sentence is the first line of the section, so "the §10 canonical input" refers to the section the reader is already in. The definition is the unnumbered **Signed input** paragraph immediately below, which calls that byte string the "canonical form" and never gives it a number to cite. §10.2 then points back at "§10" for the same paragraph. Point both sentences at the Signed input paragraph, and use one name for it. The Conformance page's link to §10 is a real cross-reference and can stay. Related, not this wording: [#195](https://github.com/GenAI-Security-Project/agent-control-standard/issues/195) (REQUIRED in this sentence, "field-optional" in §10.1).

---

## Cite composite ML-DSA for hybrid signatures instead of defining the encoding

**Where:** `docs/spec/instrument/specification.md` §10.2; the hybrid rows in the §10.1 registry (`ML-DSA-65+ECDSA-P256`, `ML-DSA-65+RSA-PSS-SHA256`); the same encoding copied into [`request-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/request-envelope.json) `Signature.algorithm`

**Notes:** §10.2 defines cryptography. It specifies `len(pqc_sig) || pqc_sig || len(classical_sig) || classical_sig` with 4-byte big-endian lengths, base64 on the wire, and verification of both components over the §10 canonical input, for any algorithm of the form `<PQC>+<CLASSICAL>`. ACS should refer to [draft-ietf-lamps-pq-composite-sigs](https://datatracker.ietf.org/doc/draft-ietf-lamps-pq-composite-sigs/) (Composite ML-DSA) and stop there. That draft serializes a signature as `mldsaSig || tradSig` with no length prefixes, because the ML-DSA component is fixed length, and it defines the message construction, the component algorithm pairs, and the rule that the composite fails if either component fails. The two registry names are not algorithms in that draft: the ML-DSA-65 / ECDSA P-256 composite is `id-MLDSA65-ECDSA-P256-SHA512`, and the ML-DSA-65 / RSA-PSS composites are SHA-512 with a fixed modulus size, not `ML-DSA-65+RSA-PSS-SHA256`. Use the draft's identifiers, and say the `value` field carries that composite signature. Delete the length-prefix rule and the "both components over the canonical input" rule from the prose and from the schema description. Related, not the same ask: [#17](https://github.com/GenAI-Security-Project/agent-control-standard/issues/17) (what a verifier does when one component fails). The draft already states that.

---

## Give the §10.3 opener a verb

**Where:** `docs/spec/instrument/specification.md` §10.3, the first sentence

**Notes:** Editorial. §10.3 opens with "`request_id` (UUID), `timestamp` (ISO 8601), and optional `nonce` (16–64 bytes)." That is a noun list. It names three envelope fields and their types and never says what the reader is to do with them. The requirements begin in the next sentence. Make the list a sentence: these fields are the replay inputs. Their types are already in [`request-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/request-envelope.json) (`request_id` as a UUID, `timestamp` as a date-time, `nonce` as a string of length 16–64).

---

## Tighten the recommended clock-skew window

**Where:** `docs/spec/instrument/specification.md` §10.3 (`skew_window_ms`, RECOMMENDED default 300000); [`handshake.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/handshake.json) `ServerHello.skew_window_ms` (`default: 300000`)

**Notes:** 300000 ms is five minutes, and the check applies in both directions, so a request timestamp may be five minutes early or five minutes late. That is a large freshness bound for a signed request. NTP is ubiquitous in modern deployments, and the residual error on an NTP-synchronized host is seconds. The schema description already tells deployments with well-synchronized clocks to tighten the window, which treats the common case as the exception. Make the recommended default the NTP case. Related, not the same ask: [#140](https://github.com/GenAI-Security-Project/agent-control-standard/issues/140) (the reference Guardian does not enforce the window at all).

---

## State platform portability as a design goal, not a 2119 MUST

**Where:** `docs/spec/instrument/specification.md` §11, the opening sentence ("ACS MUST be deployable across IDE, SaaS, on-prem on Linux/Windows/macOS/mobile/browser.")

**Notes:** An RFC 2119 MUST is a requirement on an implementation of this specification. This sentence is a requirement on ACS itself, and on every future version of it: the protocol may not gain a feature that cannot run in an IDE, a SaaS deployment, on-prem, or on Linux, Windows, macOS, mobile, and browser. That is a design goal for the editors. A deployment of v0.1 cannot conform to it or violate it. State the portability goal in plain prose.

---

## Drop the requirement that every resource identifier is a URI

**Where:** `docs/spec/instrument/specification.md` §11 ("Resource identifiers MUST use URI form"); the `resource` string on Intent capability tuples (`session-start.json`, `agent-trigger.json`, `subagent-start.json`, `ask-details.json`); the worked example in `docs/topics/ACS_in_action_example.md` (`email.send` with `resource: "manager@example.com"`)

**Notes:** The bullet does not say which identifiers it governs, and a MUST that every resource an agent acts on is a URI is not realistic. The spec's own example uses `manager@example.com`, an address. Capability `resource` is an unconstrained string. Files and HTTP endpoints already have `file` and `https`. An editor selection, an unsaved buffer, a database row, a process, a registry key, a conversation, and an in-process function have no URI form in use. `posix:///` in the same bullet is not a registered URI scheme. Name the fields that are URIs, such as an Inspect `endpoint` or `path`, or drop the MUST. Related, not the same ask: [#295](https://github.com/GenAI-Security-Project/agent-control-standard/issues/295) (scope checks on the resolved form of an identifier).

---

## The agent-layer prompt treats all input as data

**Where:** `docs/spec/instrument/specification.md` §12.2 ("Prompt MUST treat untrusted data as data, not instructions. Untrusted fields MUST be wrapped/quoted."); the §7.2 default that marks `user_input` and `system` trusted

**Notes:** The bullet only demotes untrusted data. Under §7.2 that leaves `user_input` and `system` free to be read as instructions, and a deployment may mark still more origins trusted. Trust is a channel label. It is not permission to obey the text. A user message, the Observed Agent's system content, and a tool result are all input to this layer. The instructions are the layer's own prompt. Say that all input is data, and that every input field is wrapped or quoted, whatever `trust` says.

---

## Justify why the agent layer must not see policy code, or drop the rule

**Where:** `docs/spec/instrument/specification.md` §12.2 ("MUST NOT have access to deterministic-layer policy code"); the same claim in §2 (the diagram sees "no policy code") and `README.md` ("that LLM never reads the policy source")

**Notes:** The rule is stated three times and never justified. Nothing says what goes wrong if the agent layer can read the policy: leakage of a confidential rule, the model arguing with the rule, or something else. "Access" and "policy code" are also undefined. The source text, the compiled bundle, the one rule that fired, and a `policy_references` entry are different things, and a delegated decision may need the rule it is applying. The intermediate output is the only channel §12.2 gives the layer, so forbidding policy code may forbid putting that rule in the handoff. Say which harm the ban prevents and which artifacts it covers, or drop the MUST. The handoff itself is the separate note on §2 and §12.1–12.2.

---

## (Title of next comment)

**Where:**

**Notes:**
