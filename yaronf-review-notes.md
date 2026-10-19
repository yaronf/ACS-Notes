# Documentation review notes

Working notes from reviewing the local docs preview (`http://127.0.0.1:8000/`).
Each `##` heading is one comment; titles are issue-ready.

**Open-issue check:** Before keeping a comment, search open issues on [`GenAI-Security-Project/agent-control-standard`](https://github.com/GenAI-Security-Project/agent-control-standard/issues). Drop the comment if an open issue fully covers it (same ask). Related-but-narrower issues do not count as duplicates.

Checked against open issues (2026-10-08): no comment fully duplicates an open issue. Closest overlaps (kept): large-payload note ↔ [#155](https://github.com/GenAI-Security-Project/agent-control-standard/issues/155) (truncation only); Approver/ASK clarity ↔ [#175](https://github.com/GenAI-Security-Project/agent-control-standard/issues/175) / [#51](https://github.com/GenAI-Security-Project/agent-control-standard/issues/51); MODIFY/ASK on handshake ↔ [#147](https://github.com/GenAI-Security-Project/agent-control-standard/issues/147) (substitution representation, not ClientHello declaration); scope/audience ↔ [#133](https://github.com/GenAI-Security-Project/agent-control-standard/issues/133) (FAQ page, not agent-class scope).

## Top 5

Curated as we go; re-rank when a new finding outranks an entry below.

1. [Define ACS scope and target audience across agent classes](#define-acs-scope-and-target-audience-across-agent-classes) — the standard never says who it is for.
2. [Call the HMAC baseline a MAC, and justify why not ECDSA-P256](#call-the-hmac-baseline-a-mac-and-justify-why-not-ecdsa-p256) — the Core integrity floor is misnamed, and the choice of algorithm is unexplained.
3. [Justify hook negotiation / out-of-band capability vs normative emit-when-applicable](#justify-hook-negotiation--out-of-band-capability-vs-normative-emit-when-applicable) — Core should not need an out-of-band table of which hooks are live.
4. [Put MODIFY (and ASK) capability on the handshake wire](#put-modify-and-ask-capability-on-the-handshake-wire) — the same capability is left off the wire.
5. [Clarify what control paradigms mean for implementers](#clarify-what-control-paradigms-mean-for-implementers) — IBAC, FIDES, CaMeL, and AARM read as a product matrix. They are optional citation styles.

---

## Add A2A hook pages to the MkDocs nav

**Where:** `mkdocs.yml` nav; source pages under `docs/spec/instrument/a2a/hooks/`

**Notes:** The MkDocs build warns that these pages exist and are absent from the nav. A reader of the published site cannot reach them:

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

**Notes:** The leaf label "ACS specification" sits two levels under the top-level "Specification" section. The path reads as the specification pointing at itself. Rename the leaf to the page's job under Instrument, for example the protocol and hooks contract.

---

## Define ACS scope and target audience across agent classes

**Where:** Top-level docs (landing / About / Topics intro); no explicit audience statement found

**Notes:** "Agents" in ACS covers several deployments: enterprise production agents, coding agents, and personal agents. A reader cannot tell which class the standard is written for, or whether all three are in scope. The scope statement should name each class and what ACS claims to cover for it.

`docs/README.md`, `docs/about.md`, and the pillar introductions frame ACS around the enterprise that welcomes an agent in. They do not enumerate the classes.

Mandatory ASK does not by itself make coding agents the audience. A Guardian MUST support ASK. An approver MAY be a human, an agent, or a service (§9). §9.2 substitutes DEFER or DENY for an approver-incapable client, including headless automation. An interactive coding agent is a natural consumer of ASK. It is not a declared primary audience. The enterprise framing and the gravity of interactive approval are two reasons the scope statement is missing.

---

## Balance Inspect transparency against attacker reconnaissance

**Where:** `docs/README.md` — Trustworthy Agents blurb, Inspectable paragraph ("What are the services behind them, and which data they can access.")

**Notes:** The blurb presents that visibility as purely desirable. The same map of services and data access is useful to an attacker. The introduction should state the tradeoff.

---

## Call the HMAC baseline a MAC, and justify why not ECDSA-P256

**Where:** `docs/acs.md` ("baseline HMAC-SHA256 signature"); also `docs/spec/conformance.md`, `docs/spec/instrument/specification.md` §10 (registry correctly labels HMAC as "Symmetric MAC" but the surrounding prose still says "signature")

**Notes:** HMAC-SHA256 is a MAC. Calling the Core baseline a signature hides two limits the same section already admits: it provides no non-repudiation, and it does not bind a compromised Guardian, because the Guardian holds the key.

The mandatory algorithm is a separate question. ECDSA-P256 is the natural classical choice, and it is already OPTIONAL in the §10.1 registry. HMAC is a reasonable same-host shortcut. If HMAC remains the Core baseline, the text should say why: adoption, or simplicity, or both.

---

## Mention ECDSA-P256 in the ACS-Crypto overview blurb

**Where:** `docs/acs.md` — ACS-Crypto profile bullet ("HMAC-SHA256 baseline, ML-DSA-65 / SLH-DSA-128s for PQC, hybrid composites…")

**Notes:** The overview omits classical asymmetric algorithms. `ECDSA-P256` is OPTIONAL in the §10.1 registry, and the registry calls it the algorithm with the strongest current ecosystem support. In the blurb it appears only inside a hybrid composite. A reader of the profile list concludes that ACS-Crypto is HMAC plus post-quantum algorithms. Name ECDSA-P256 beside the post-quantum algorithms. RSA-PSS belongs in the same sentence if it stays in the registry. The open question is whether P-256 should be the baseline.

---

## Clarify how Runtime Hooks relate to ACSInstrument

**Where:** `docs/acs.md` — Trustworthy agents are › Instrumentable › "Runtime Hooks" example (follows the earlier "Init agent with ACS" / `ACSInstrument` snippet)

**Notes:** A new reader has just seen an agent wrapped in `ACSInstrument`. The next example registers `toolCallRequest` hooks by hand and calls `guardian.evaluate` itself. The text should say what the wrapper does by itself, and when hand registration is an additional control.

The Python example branches only on `DENY` and `MODIFY`. The dispositions also include `ALLOW`, `ASK`, and `DEFER`. An unexpected `action` has no path. In Python that path is `else`, or `case _:` in a `match`. The TypeScript `switch` has the same gap.

---

## Clarify that AgBOM serializations are not required to be signed

**Where:** Inspect definition — `docs/spec/inspect/README.md` (and related CycloneDX/SPDX/SWID extension pages)

**Notes:** CycloneDX treats a signature on a BOM as optional. ACS-Inspect should say the same: a signed serialization is not required for conformance. Integrity for Inspect is the ACS envelope on `agbom/snapshot` and `agbom/changed`, and the SessionContext chain.

---

## Put MODIFY (and ASK) capability on the handshake wire

**Where:** `docs/spec/instrument/specification.md` §6.5 (MODIFY-incapable clients); same pattern in §9.2 (approver-incapable clients); contrast §4 Capability Negotiation Handshake

**Notes:** §6.5 tells the Guardian to learn, out of band, that a client cannot apply MODIFY. The signals it lists are agent identity, a policy keyed on `agent_id`, organizational configuration, and "any other signal." It then says v0.1 does not put the declaration on the wire. ACS-Core already requires a capability handshake, with `methods_implemented` and `profiles_supported`. The client's ability to apply MODIFY belongs in ClientHello. §9.2 has the same gap for ASK.

---

## Rename the Instrument pillar to name what it does

**Where:** Pillar naming across the docs (e.g. Instrument / Trace / Inspect); `docs/spec/instrument/`, nav, conformance, intros

**Notes:** The three pillars are Instrument, Trace, and Inspect. Trace and Inspect name outcomes. Instrument names the mechanism, the hooks, rather than the job, which is the runtime permit, deny, or modify of an agent action. A verb such as Govern, Police, or Control would make the three names parallel. Examples, not a required choice.

---

## Include SPIFFE (and future WIMSE) in identity type examples

**Where:** `docs/concepts/identity.md`, `docs/topics/core_concepts.md`, `docs/spec/instrument/specification.md` §11 (identity descriptor `type` examples: `posix_uid`, `oauth_subject`, `cert_subject`, …)

**Notes:** The example `type` list omits workload identity. A SPIFFE ID is the natural type for an agent or workload principal. WIMSE identifiers are the later form of the same idea. `docs/identity/standards.md` already proposes WIMSE and SPIFFE as the recommended model. Add `spiffe_id` to the illustrative set, with a forward pointer to WIMSE, so the core identity text matches that workstream.

---

## Consider RFC 9493 Subject Identifiers instead of ad-hoc oauth_subject

**Where:** `docs/concepts/identity.md` (and related identity type examples); `docs/spec/instrument/specification.md` §11 (identity descriptor `type`: `posix_uid`, `windows_sid`, `oauth_subject`, `cert_subject`, …); Identity workstream descriptor schema

**Notes:** [RFC 9493](https://www.rfc-editor.org/rfc/rfc9493.html) defines subject identifiers for security event tokens: a typed JSON object, a `format` discriminator (`iss_sub`, `email`, `uri`, `did`, `opaque`, and others), and an IANA registry. `iss_sub` names an OAuth or OIDC principal by issuer and subject. That pair is what makes the identifier unique. ACS's illustrative `oauth_subject` plus a bare `sub` does not. The format is in use through OpenID Shared Signals, CAEP, and RISC, for example Okta SSF. It is not yet a universal OAuth wire encoding. ACS should align its identity descriptors with that registry, and use `uri` or `did` for SPIFFE and WIMSE, instead of growing a parallel type list.

§11 is that list in the specification. The discriminator is `type`, the examples are `posix_uid`, `windows_sid`, `oauth_subject`, and `cert_subject`, and the schema is deferred to the Identity workstream. The same alignment applies there.

---

## Identity "verified credential" claim has no ACS wire support (e.g. PoP)

**Where:** `docs/concepts/identity.md` ("An asserted identity is weaker than one bound by a verified credential or signature"); related trust-basis language in `docs/topics/core_concepts.md`

**Notes:** The sentence says ACS can tell an asserted identity from one bound by a verified credential. On the wire, an identity descriptor is a typed claim. There is no field for proof of possession, DPoP, an mTLS binding, or an SVID presentation. Authentication remains deployment-defined. ACS-Crypto attests envelopes. It does not attest a principal credential. The identity workstream requires DPoP or mTLS compositionally, in `docs/identity/standards.md`. The Core identity descriptor does not say how a Guardian learns that an identity was verified that way. Add that, or soften the sentence so it does not read as a Core feature.

---

## Replace leftover "ASOP" references with ACS

**Where:** `docs/topics/core_concepts.md` diagram (`docs/assets/agent_env.png` — edge labeled ASOP between Observed Agent and Guardian); `docs/spec/instrument/a2a/extend_a2a.md` ("uses ASOP as a transport"); `docs/spec/trace/OCSF/implementation_examples.md` (`ASOP Security Layer`, `unmapped.asop.*`)

**Notes:** "ASOP" is never defined. It reads as a leftover project name, next to the older AOS branding, in examples and in the Core Concepts diagram. Rename those occurrences to ACS. If the OCSF namespace stays, rename it to `unmapped.acs.*`. A new reader should not have to hunt for a fourth acronym.

---

## ACS-in-action sequence: DENY is legal at every allow, and AgBOM is mis-ordered

**Where:** `docs/topics/ACS_in_action_example.md` — Sequence mermaid

**Notes:** The walkthrough shows `allow` on a permit path. A reader can take that as the only legal verdict. DENY is legal at each decision the diagram shows.

* `agbom/snapshot` may refuse a banned component.
* `sessionStart` may refuse the session.
* `userMessage` allows ALLOW, DENY, and MODIFY.
* `toolCallRequest` allows the full vocabulary.
* `toolCallResult` allows ALLOW, DENY, and MODIFY.

Say so on the diagram, or add a deny branch. The page later shows deny-shaped examples only for the tool-call paradigms.

The diagram also emits `agbom/snapshot` before `sessionStart`. The normative order is after `sessionStart` and before the first content-bearing hook. That order is in `hooks.md` and the Inspect README.

---

## Specify large-payload handling: stream, truncate, compress, or refuse — with an indicator

**Where:** Wire format / Instrument — `docs/spec/instrument/specification.md` §3 (streaming deferred to v0.2), §4 `max_payload_size_bytes`; no truncation/compression fields found in schemas

**Notes:** A hook body can be very large. Tool results, messages, and skill definitions are the obvious cases. v0.1 defers streaming of hook traffic to v0.2. It also defines no truncation mode and no flag that a payload was truncated. ClientHello carries `max_payload_size_bytes`, which is the Observed Agent's willingness to send. There is no matching Guardian limit, no refuse error, and no alternative that sends a hash or a summary. Compression is likewise unspecified. HTTP may compress out of band. The ACS envelope does not negotiate it.

The specification should say which of these an oversized body takes: refusal, a hash or summary, compression, or a later stream. It should also say how the Guardian knows it did not see the full body.

In v0.1 the Guardian still needs its own receive limit, for example on ServerHello. Otherwise it cannot bound memory, and a large body is a denial of service. Related, and narrower: [#155](https://github.com/GenAI-Security-Project/agent-control-standard/issues/155) covers truncation only.

---

## Document authorization paradigms without forcing them into Concepts

**Where:** IBAC/FIDES/CaMeL/AARM used across Instrument and Topics; no dedicated page. Concepts README altitude rule: cross-cutting → Concepts, mechanism → pillar

**Notes:** A reader needs a definition of the named paradigms. IBAC, FIDES, CaMeL, and AARM are not common OWASP terms, and the docs expand them only lightly. Authorization and policy evaluation are an Instrument mechanism. A Concepts page would break the altitude rule Concepts exists to enforce: cross-cutting material goes to Concepts, and a single-pillar mechanism stays in the pillar. Put the paradigm-to-wire mapping, the engines (OPA and Cedar), and the statement that these are optional citation styles on an Instrument page, or on a Topics page that points at §6.2 and §12. Leave Intent, Capability, and Trust in Concepts. Those are the invariants the paradigms consume.

---

## Stop restating structured decision fields in reasoning examples

**Where:** `docs/topics/ACS_in_action_example.md` (e.g. FIDES deny `reasoning`); also other paradigm examples there. Spec: `docs/spec/instrument/specification.md` §6.1

**Notes:** The example `reasoning` strings repeat fields that already have a place: `reason_codes`, `policy_references`, `policy_data`, and `cited_provenance_ids`. §6.1 defines `reasoning` as one human-renderable explanation. It tells a UI or a meta-policy to switch on `reason_codes` rather than parse the prose. An example should show short text for a person, or omit the field. A second encoding of the structured payload teaches the reader to dump structure into `reasoning`.

---

## Clarify what control paradigms mean for implementers

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite"; also Instrument §6.2 (and whatever Instrument/Topics page eventually defines paradigms — not Concepts)

**Notes:** The section presents IBAC, FIDES, CaMeL, and AARM as features a Guardian might be expected to ship. An implementer cannot tell whether a Guardian must implement all of them, some of them, or none. The text also does not say whether they are negotiated, or whether they are normative profiles or external architectures that ACS accommodates.

The rest of the docs answers that, and this section should lead with the answer. The wire is paradigm-neutral: `policy_data`, `reason_codes`, and `policy_references`. The handshake negotiates ACS profiles, for example `acs-provenance` for an information-flow style, and it does not negotiate paradigm names. Pure IBAC needs no Provenance profile. No field declares the enforcement paradigm. ClientHello and ServerHello do not. `sessionStart` has `policy_mode`, and that value is a `scope_mode`, not IBAC or FIDES. A sentence such as "Intent is optional but normative when IBAC is the enforcement paradigm" hangs a MUST on a choice the deployment never declares.

Lead with that story: an optional policy style, and cite what fired. Link the source papers. Say how complete each mapping is. The alternative is to declare the paradigm on the wire or in a profile. Rephrase those MUSTs as "when the deployment uses Intent or an IBAC-style policy."

---

## Show conventional authz in ACS-in-action, not only research paradigms

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite" (IBAC, FIDES, CaMeL, AARM); contrast Instrument §12.1 (OPA/Rego as v0.1 reference, Cedar as v0.2)

**Notes:** The worked example leads with research paradigms and never shows a conventional authorization rule: allow or deny on identity, role, and resource, or a plain OPA or Cedar rule. A reader infers that ACS is for FIDES, CaMeL, and AARM only. Add at least one ordinary example, and lead with it. The deterministic layer's starting reference is OPA/Rego. The paradigms are optional citation styles.

---

## Clarify whether Approver is optional and who resolves ASK

**Where:** `docs/concepts/agents.md` (Approver); Instrument §9 / §9.2; ASK as Core MUST-support disposition

**Notes:** The text mixes two models, and an implementer cannot tell which one is required.

* Concepts and §9: the Guardian consults a third-party Approver at `ask_details.approver` while the Observed Agent pauses.
* §9.2: some Observed Agents route ASK and need approver UX, which implies the main agent resolves ASK locally, for example an IDE user.

Say whether the Guardian MAY return `ask` with no designated Approver. Say who presents the approval UI. Say when the Approver may be omitted entirely: never raise ASK, or substitute DEFER or DENY only. Related, and narrower: [#175](https://github.com/GenAI-Security-Project/agent-control-standard/issues/175) and [#51](https://github.com/GenAI-Security-Project/agent-control-standard/issues/51).

---

## Rewrite the opaque "trust basis, not that it arrived" sentence

**Where:** `docs/concepts/agents.md` — Trust between agents: "How much weight a given fact carries depends on its trust basis, not on the fact that it arrived."

**Notes:** The last sentence depends on the reader already knowing [Trust basis](docs/concepts/trust.md). Say the point in plain terms. Arrival on the wire does not make a claim true. Reliance follows how the fact was produced: asserted, attached by the framework, or cryptographically attested. Point at Trust basis. The punchline should not depend on the words "weight" and "trust basis."

---

## Clarify Guardian identity vs the policy (and policy-author)

**Where:** `docs/concepts/identity.md` — "Guardian identity: which policy authority is deciding."; also `docs/topics/core_concepts.md`

**Notes:** The wording makes Guardian identity sound like the policy. Elsewhere ACS keeps three parties: the Observed Agent, the Guardian, and the policy author. Conformance already says the policy author is not the Guardian. Guardian identity is the deciding party, its own principal. It is not the policy document, and it is not the policy author. Say what, if anything, carries that identity on the wire in v0.1.

---

## Fix broken lead sentence on Identity overview

**Where:** `docs/identity/overview.md` — "There are 5 unique identity challenges that this standard aims to solve, where existing use cases that traditional identity models cannot support."

**Notes:** Editorial. The clause after the comma does not parse. Rewrite it as prose: five challenges that traditional identity models cannot support for agent use cases. Drop the filler "unique" and "aims to solve." STYLE.md already asks for that.

---

## Drop the false "build-time vs execute-time supply chain" claim

**Where:** `docs/identity/overview.md` — "The supply chain you signed off on at build time is not the supply chain that executes."

**Notes:** The sentence is misleading. Tools, the runtime environment, and often the prompts are stable. The supply chain signed at build time is often the one that executes. The point worth keeping is non-deterministic control flow at runtime: which tool, in which order, with which arguments. Rewrite the sentence without the supply-chain metaphor.

---

## Do not imply agentic identity stops the "valid token, bad outcome" failure class

**Where:** `docs/identity/overview.md` — "Agent systems are introducing a new class of failures where every token is valid, every API call is authorized, and the outcome is still a security incident." (and the Unit 42 example that follows)

**Notes:** The failure class is real. Agentic identity does not close it. Identity supports governance and forensics: who acted, under which chain, and under which intent. Prompt injection and similar attacks still succeed against valid credentials. Say that, so a reader does not treat naming and chaining as the fix.

The next two sentences do not follow. The runtime could not distinguish user intent from adversarial instructions. That is a control and intent problem. The punchline calls it the central identity challenge. Point the punchline at Instrument and Intent, or say what identity-shaped gap the Unit 42 example is meant to show.

---

## Restructure the five identity-challenges table for readability

**Where:** `docs/identity/overview.md` — "The 5 Unique Runtime Identity Challenges for Agents" table

**Notes:** Editorial. Each cell is a long paragraph: the threat, the standards gap, the desired outcome, and the status, in one row. The table does not scan in MkDocs. Give each challenge one short row and a link to a subsection. A definition list or cards would serve the same purpose. Keep the comparison surface short.

---

## Stop calling the identity challenges "unique"

**Where:** `docs/identity/overview.md` — "5 Unique Runtime Identity Challenges" (heading and body); at least Over-Privilege and Token Theft Resistance are long-standing IAM problems

**Notes:** Drop the uniqueness claim. The challenges are real. Over-privilege and token theft, at least, are long-standing identity problems. They are not unique to agents.

---

## Separate chain-integrity forensics from chain-based policy enforcement

**Where:** `docs/identity/overview.md` — Chain Integrity challenge (and related identity-standards prose)

**Notes:** Naming and verifying a delegation chain is useful for forensics. It is much less clear that a real policy can be driven by a long chain. A human-authored rule does not express conditions over a deep chain, and other authorization systems rarely do so at runtime. Split the ask. One part is audit and reconstruction. The other is enforcement. "Verify the full chain at every hop" is not an obvious policy input until the text shows how a Guardian would consume it.

---

## Correct the false claim that API keys are always bearer secrets

**Where:** `docs/identity/overview.md` — Token Theft Resistance row: "API keys are bearer secrets by definition."

**Notes:** The sentence is false as a universal claim. Many API keys and access-key pairs are used to MAC or sign a request. AWS SigV4 is one example. They are not presented as bearer credentials. Say that a *bearer* API key, or an opaque token, is sufficient for whoever holds it. Do not flatten every API-key scheme into that class.

---

## Do not call GNAP and AMR "broadly adopted"

**Where:** `docs/identity/standards.md` — Existing Standards intro: "The following standards are broadly adopted…"

**Notes:** The blanket claim is false for at least two entries. GNAP (RFC 9635) is published and has little real-world deployment. AMR (RFC 8176) is an OIDC authentication-method claim, and it is not a foundation of modern identity. Soften the introduction, or split it into established and emerging. Do not put GNAP and AMR in the same adoption bucket as OAuth 2.0 and OIDC.

---

## Do not claim token issuance cannot support per-action authz

**Where:** `docs/identity/standards.md` — Why OAuth Alone Doesn't Meet Agent Needs table: "No token-issuance standard can patch this; it requires inline behavioral verification at action time."

**Notes:** The claim is too broad. OAuth can carry fine-grained authorization. Rich Authorization Requests are one example, and this page already says ACS requires them. Token issuance can express an action-scoped policy. The gap against a hijacked client is detecting misbehavior and verifying intent at runtime. Narrow the sentence to that. The same table lists OAuth extensions as mitigations in other rows. This row alone reads as if OAuth can never be part of a solution.

---

## Short TTL alone is not enough for high-risk actions

**Where:** `docs/identity/standards.md` — refresh-token mitigation row and Token lifetimes summary: proposed "60 s for high-risk actions"

**Notes:** A 60-second ceiling still leaves a wide window. A minute is enough time to do harm. A short lifetime is not, by itself, the control for a high-risk action. Where the risk warrants it, the credential should be single-use. Say that. Do not treat an aggressive TTL as sufficient.

---

## Do not treat fail-open as an open peer option to OAuth/AIMS

**Where:** `docs/identity/standards.md` — IdP reachability row: "Neither OAuth nor AIMS specifies a failure-mode contract." → open work item "fail-closed vs. fail-cached vs. tier-dependent policy."

**Notes:** OAuth and AIMS do not leave a hole labeled fail-open. The community reads them as fail-closed: deny when the identity provider cannot authorize. An ACS work item that treats fail-closed, fail-cached, and tier-dependent as peers understates the risk of fail-open here.

Fail-open can make sense when the policy decision point is outside the attacker's reach. Unavailability is then mostly an operations failure. That condition does not hold for the Guardian. The Observed Agent already has a wide attack surface against it, including prompt injection. A simple denial of service induces a decision failure, and fail-open then lets the action through. The default posture should be fail-closed. A fail-open or fail-cached path needs its own justification.

Related, and narrower: [#32](https://github.com/GenAI-Security-Project/agent-control-standard/issues/32) (refusals should fail closed) and [#37](https://github.com/GenAI-Security-Project/agent-control-standard/issues/37) (the shape of a fail-open audit event).

---

## Immutable Intent needs a real integrity story, not only an invariant

**Where:** `docs/concepts/intent.md`; `docs/spec/instrument/specification.md` §8.4; identity pages that cite "immutable intent"

**Notes:** The specification states an immutability rule. `Intent.parsed` MUST NOT be widened by the LLM, by tool outputs, or by an untrusted channel. It may grow only through ASK and `intent_extension`. The framework MUST ignore or reject an attempt. That is an invariant. It is not a mechanism.

The text does not say how Intent is sealed when it is established: who parses `raw` into `parsed`, or how `parser_provenance.origin: user_input` is verified rather than asserted by the sender. It does not say how a host or Guardian stops the Observed Agent from supplying or rewriting Intent before commit. It does not say how to stop a reframing that never mutates the capability list. Related, and narrower: [#149](https://github.com/GenAI-Security-Project/agent-control-standard/issues/149) (canonical Intent when `sessionStart` carries none).

---

## Intent is on the wire; stop calling it "not a wire format"

**Where:** `docs/concepts/intent.md` opener ("Intent is a governance concept, not a wire format"); hook schemas that embed Intent (`session-start.json`, `agent-trigger.json`, `subagent-start.json`); §8.4

**Notes:** The opener contradicts the schemas. Intent is carried on the wire: `raw`, `parsed`, `parser_provenance`, and `scope_mode`. Extract a shared Intent schema and document the on-wire types in prose. At minimum, `raw` is a free-text string and `parsed` is a capability list. Today only the JSON Schema says `raw` is a string, and the concept page never does. The alternative is to drop "not a wire format" and say Intent is a governance concept with a wire representation.

---

## Document goal→capability derivation for Intent.parsed

**Where:** `docs/concepts/intent.md` (`Intent.parsed` = "the set of things the agent is permitted to do"); also `docs/topics/ACS_in_action_example.md` (example Intent `["summarize Project X"]` vs schema capability tuples)

**Notes:** In a real chat, a person states what the agent is required to do, for example "summarize this document." The person does not state an exhaustive permission set: read this document, read related documents, write a summary, fetch from the network. Those permissions are derived. ACS never says who performs the derivation: the host, the Guardian, or a separate parser. It does not say which inputs are used, how over- and under-approximation are handled, or how the derivation is audited beyond `parser_provenance`. The worked example stores a goal string as Intent. The wire schema expects `{tool, operation, resource}` tuples. Specify the derivation contract, or stop presenting `Intent.parsed` as a permission set the user authors directly.

---

## Define a capability registry (or admit policy is deployment-local)

**Where:** `docs/concepts/capability.md`; §11 ("Capability vocabulary uses abstract names…"); AgBOM / `tool-call-request.json` (`capability` as free string with examples)

**Notes:** ACS does not define a concrete capability set, for example `filesystem.delete`, and it provides no registry. The names in the text are illustrative. Without a shared vocabulary, a Guardian, an `Intent.parsed` list, and a tool declaration cannot interoperate across vendors. The same action can be labeled differently, and a policy bundle is not portable. Ship a registry: a core set of identifiers, plus a registration and extension process. The alternative is to say explicitly that IBAC interoperability is deployment-local string matching.

The shapes also drift. Concepts and examples use capability identifiers. The `Intent.parsed` schema uses `{tool, operation, resource}` tuples. This note is about the permission vocabulary. The tool-call action syntax is a separate note.

---

## Untangle Capability vs agent_capability in the concepts glossary

**Where:** `docs/concepts/skill.md` ("A skill differs from an `agent_capability`…"); `docs/concepts/capability.md`; concepts README

**Notes:** In the glossary, "Capability" means two things.

* On the Capability page it is an abstract permission identifier, for example `filesystem.delete`.
* In the AgBOM it is an `agent_capability` component, a passive grouping of tools, MCP servers, and peers.

`skill.md` contrasts a skill with `agent_capability` and links that term to `capability.md`, which mostly defines the permission identifier. A reader cannot tell which meaning the eight-concept list intends. Keep Capability for the permission. Give the AgBOM type a name that does not collide. Fix the cross-links.

---

## Rewrite "trust launder" jargon in Provenance/Trust openers

**Where:** `docs/concepts/provenance.md` opener ("lets trust launder silently"); also `docs/concepts/trust.md` ("trust laundering up the spectrum")

**Notes:** "Launder" is an opaque metaphor for a first sentence. Say what happens. If provenance or the trust basis is dropped at a boundary, untrusted or merely asserted data can be treated as trusted on the far side. Keep "laundering" only if it is glossed once. The Provenance opener should not depend on the reader already knowing the phrase from the Trust page.

---

## Separate channel/metadata integrity from content trust on the Trust Basis spectrum

**Where:** `docs/concepts/trust.md` — spectrum ("Each rung is a stronger ground for reliance")

**Notes:** The ordered spectrum invites a reader to treat two questions as one. Framework-deterministic provenance, for example "we read this from the database," strengthens confidence in the origin label. It does not raise trust in text that arrived from an untrusted source. A signature or a hash authenticates the emitter and the integrity of the bytes. Signing untrusted content does not make the substance more believable. Rewrite "Deterministically attached" and "Cryptographically attested" so they are about ACS metadata and channel claims: provenance fields, envelopes, and the audit chain. The invariant that rungs do not collapse does not repair rungs that are already described as levels of content trust.

---

## Clarify that profile "layers" are declared, not proven

**Where:** `docs/spec/conformance.md` — diagram intro: "Each layer is a self-declared claim from the handshake, not a verified fact."

**Notes:** The wording is cryptic. It is not a protocol flaw. A reader of the stack diagram hears "a layer is a claim from the handshake," and that sentence does not parse. The intended meaning, said more clearly later, is that `profiles_supported` and `profiles_accepted` are advertisements. That pattern is ordinary capability negotiation. Strong verification of a peer's capabilities is rare, and systems usually rely on an implicit trust assumption. Rewrite the caption in plain terms: these are optional conformance profiles, and the handshake names are advertisements. Keep the "Who verifies" paragraph for a buyer. Do not dramatize ordinary self-declaration on the figure.

---

## Call the wire a JSON-RPC 2.0 profile — there are no real JSON-RPC extensions

**Where:** `docs/spec/conformance.md` / §3 ("JSON-RPC 2.0 with ACS extensions"); `request-envelope.json` / `response-envelope.json` descriptions

**Notes:** As schematized, ACS does not extend JSON-RPC 2.0 at the protocol layer. The top-level members are the standard ones: `jsonrpc`, `method`, `id`, and `params` or `result` or `error`. ACS fields such as `acs_version`, `signature`, and the decision sit inside `params` or `result`. That is ordinary application payload. What ACS adds is a constrained profile: no notifications, method namespaces, a required `params` shape, `additionalProperties: false`, and optional batching. Prefer that wording over "with ACS extensions."

§3 claims one non-conforming extension, a top-level `acs_version`. The schema contradicts that sentence. That defect is already [#194](https://github.com/GenAI-Security-Project/agent-control-standard/issues/194).

---

## Justify hook negotiation / out-of-band capability vs normative emit-when-applicable

**Where:** `docs/spec/conformance.md` (Guardian determines subagent-emission capability out-of-band); §4 `methods_implemented` / `methods_evaluated`; similar ASK/MODIFY "deployment-defined means" precedents

**Notes:** The specification asks the Guardian to know which hooks are active, and it does not say why. The alternative is a rule on the harness: if the framework can observe or spawn an event, it MUST emit the corresponding hook. Absence then means the event did not occur. There is no out-of-band capability table and no negotiated subset. Today the Guardian infers the client's ability from identity and policy configuration. That inference is fragile, and it duplicates what a MUST-when-applicable rule would already require.

`methods_implemented` and `methods_evaluated` need an explicit rationale, for example a partial Guardian or an optional observability hook. Otherwise drop them in favor of a fixed Core hook set plus conditional emission. Related, and about coverage detection rather than this design: [#16](https://github.com/GenAI-Security-Project/agent-control-standard/issues/16) and [#31](https://github.com/GenAI-Security-Project/agent-control-standard/issues/31).

---

## Derive signature algorithm from the key, do not trust wire `algorithm`

**Where:** §10 signature envelope `{ algorithm, value, key_id }`; hooks.md common-envelope example; `request-envelope.json` `Signature`

**Notes:** A verifier should take the algorithm from the key material looked up by `key_id`, and reject a mismatch. An explicit `algorithm` on the message is the wrong authority. An attacker claims a weaker algorithm, or the wrong algorithm, for the same key.

Prefer `key_id` and `value` only. Carry the algorithm, including a hybrid composition, in the key descriptor. If `algorithm` stays on the wire for debugging, a verifier MUST ignore it except as a check that it matches the key. Hybrids already say `key_id` resolves to a descriptor that pins both keys. That descriptor is the right place for the algorithm identity.

This does not damage the crypto agility §10.1 claims. That claim is the registry plus the handshake-negotiated set. The migration sentence already says only that set shifts, with no wire-format change. A new algorithm is a new key descriptor and a new handshake entry.

---

## Say what agentTrigger MODIFY rewrites — not vague "before activation"

**Where:** `docs/spec/instrument/hooks.md` — agentTrigger: "Guardian MAY rewrite the trigger payload (e.g. redact PII) before activation."

**Notes:** "Activation" does not say which bytes change. If MODIFY changes what the agent consumes when it starts on the trigger, say that in the same terms as `userMessage`: before delivery to the agent. If the rewrite is only for the audit record, say "audit." As written, the sentence confuses the live path with the audit trail.

---

## Make modified_content a structured payload replacement, not a string

**Where:** `specification/v0.1.0/modifications.json` (`modified_content: string`); §6.3 ("opaque replacement string")

**Notes:** A wholesale MODIFY is typed as a string. Hook payloads are structured objects. `userMessage.content` is an array of typed parts. A tool call is an object. A string cannot replace those unless the encoding is specified, and JSON stuffed into a string is not a specification. Type `modified_content` as the replacement payload object, the same shape as the hook's `params.payload`, or at least an open object. Keep `redactions` and `parameter_overrides` for a partial edit. Related host pain: [#181](https://github.com/GenAI-Security-Project/agent-control-standard/issues/181).

---

## Specify a core subset of tool-call actions, extensible later

**Where:** `specification/v0.1.0/hooks/tool-call-request.json` (`tool.name`, optional `operation`, open `arguments`, optional `raw_command`); not the capability vocabulary

**Notes:** A capability is an abstract permission label. A tool call is the concrete action: what ran, and with which arguments. ACS defines neither a syntax nor a catalog for the tool call. `name`, `operation`, `arguments`, and `raw_command` are free-form. There is no shared shape for the common cases: a shell or syscall, a file read, write, or delete, and a network connect or fetch. Put a small normative subset of those actions in the main spec, with typed arguments. Define an extension point for everything else. A full tool ontology can wait. Dotted capability examples are not that subset.

---

## preCompact DENY is not a durable policy once the window is full

**Where:** `docs/spec/instrument/hooks.md` — preCompact: Guardian MAY DENY compaction (e.g. until trusted re-grounding)

**Notes:** The hook is a realistic lineage chokepoint. `postCompact` binds `derived_from` to the union of the compacted entries. DENY is not a durable policy. `triggered_by: size_threshold` means the runtime is already out of room. Refusing compaction does not keep the uncompacted context usable. A policy that says "do not compact untrusted data" has to say what happens instead: drop specific entries, compact with the tainted lineage preserved, or end the session. An open-ended block does not. Treat size-forced compaction as inevitable. Put the control on what the summary is allowed to claim.

---

## Do not present "fabricate a chain hash" as an option

**Where:** §8.6; `docs/spec/instrument/hooks.md` subagentStop; `subagent-stop.json` — "omits the field rather than fabricate a value"

**Notes:** Omitting `final_chain_hash` when there is no session chain is the right rule. Naming fabrication as the alternative invites an implementer to consider a forged hash. A made-up value fails immediately against the published chain head, as `CHAIN_MISMATCH`. It is not a workable shortcut. Say the field is absent when the framework has no chain.

---

## Do not push skill vetting entirely outside ACS

**Where:** `docs/spec/instrument/hooks.md` skillRegister ("Marketplace pre-publication scanning and publisher reputation are out of scope"); `specification/proposals/skill-lifecycle/README.md` § Scope boundary ("the marketplace's job")

**Notes:** The stated split gives ACS only the register and load control point. Scanning and publisher reputation belong to the marketplace. That drops two things an enterprise needs. A prior scan result, or a publisher or registry decision, avoids re-analyzing the skill body on every register or load. The same artifact is how an enterprise states its own vetting policy: only digests in our registry, this publisher, this scan passed. Re-deriving trust from the raw definition each time is the cost of leaving it out.

ACS does not have to run the scanner. It does need a place to carry a prior attestation, a publisher identity, or an enterprise-registry decision into `skillRegister` and `skillLoad`, so policy can rely on it. `registration_provenance` does not fill that place. It is a normal Provenance object: `origin`, optional `source_id`, and `derived_from`. Prose says `source_id` SHOULD name a publisher, and a Guardian MAY require a signed provenance. The schema has no signature, no scan result, and no registry verdict.

---

## Do not file Inspect methods under Instrument "Supported Hooks"

**Where:** `mkdocs.yml` — Specification → Instrument → Supported Hooks (`docs/spec/instrument/hooks.md`); the same page's `agbom/snapshot` and `agbom/changed` rows

**Notes:** The nav presents one hook catalog as Instrument. The page itself says `agbom/*` are Inspect-pillar methods. The Instrument specification says they share the request envelope and the audit chain, and that they are Inspect methods. `system/ping` is the same kind of misfit: liveness, not a control hook. A shared envelope reference can stay. List the Inspect methods under Inspect. They are already summarized in `docs/spec/inspect/README.md`. "Supported Hooks" should be the control hooks.

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

## Use the JSON-RPC id as the step identifier

**Where:** [`request-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/request-envelope.json) (`id` is a string or number; `params.request_id` is a separate UUID); [`response-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/response-envelope.json) (echoed `id`, plus `result.request_id`); `docs/spec/instrument/specification.md` §10.3 (replay keys on `request_id`); [`context-entry.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/context-entry.json) (`request_hash` hashes `params`; `step_id` should equal `request_id`)

**Notes:** Replay, the audit chain, and trace can all use one identifier. Constrain the JSON-RPC `id` to a UUID that is unique within the session, and use it as the step id. A duplicate `id` is `REPLAY_DETECTED`. `step_id`, `turn_id`, `request_id_ref`, and the trace joins copy that `id`. The signature already covers it, because §10 signs the whole envelope. Drop `params.request_id` and the copy on the result. One commitment moves. `request_hash` is the hash of `params` only, so today it includes `request_id`. After the move the content hash no longer includes the step identity. The chain entry still binds it: `step_id` is covered by `entry_hash`. Say that in the `request_hash` rule. No open issue asks for this.

---

## Give system/ping its own response subschema

**Where:** [`response-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/response-envelope.json) `result` `oneOf` (`AcsResult` or `ServerHello` only) and `AcsResult.payload` (an object with no properties; the description says `system/ping` returns `{ status, echo, server_timestamp }`); `docs/spec/instrument/specification.md` §13, the Response paragraph; [`hooks/system-ping.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/hooks/system-ping.json) (request `echo` only)

**Notes:** `handshake/hello` has a named result shape, `ServerHello`. `system/ping`'s reply does not. §13 states it in prose: `decision: "allow"` and `payload` of `{ "status": "ok", "echo": "<request.echo>", "server_timestamp": "<iso-8601>" }`. The envelope description repeats that shape as an example, and `payload` accepts any object. Give the ping response its own subschema and refer to it from `result`, the way `ServerHello` is a branch of that `oneOf`. A ping result still carries `decision`, so the subschema is that payload on an `AcsResult`, with `status`, `echo`, and `server_timestamp` required as §13 describes them. Related, not the same ask: [#129](https://github.com/GenAI-Security-Project/agent-control-standard/issues/129) (the reference Guardian does not implement `system/ping`).

---

## Ship an OPA/Rego binding in v0.1, or stop calling it the starting reference

**Where:** `docs/spec/instrument/specification.md` §12.1 (OPA/Rego as "v0.1 starting reference"; the conventions sentence naming bundle layout and a baseline policy bundle) and §16 (v0.2 highlight "Cedar binding")

**Notes:** §16 schedules a Cedar binding for v0.2. A binding is the mapping from the §12.1 interface onto one engine: the input document the engine evaluates, the result it returns, and the bundle that holds the policy.

§12.1 names OPA/Rego as the v0.1 starting reference. That row is a status label. v0.1 contains no OPA/Rego binding. There is no Rego input document, no mapping from an OPA result onto the decision envelope, and no bundle layout. The same paragraph says ACS layers on a bundle layout and a baseline policy bundle. Neither is written down.

The one-sentence interface is not that binding. It names the inputs, the request envelope, SessionContext, Intent, and provenance, and the output, a decision envelope plus an optional `delegate_to`. It does not say how those objects appear inside Rego. The worked example sets `evaluator_version` to `opa-0.65`. That names a binary. Write the OPA/Rego binding as a v0.1 artifact, the same kind of artifact §16 promises for Cedar, or drop "v0.1 starting reference." The agent-layer handoff is the separate note on §2 and §12.1–12.2. No open issue asks for the binding.

---

## Make the §17.1 error object agree in the prose, the table, and the schema

**Where:** `docs/spec/instrument/specification.md` §17.1 (the opening paragraph and the registry table); [`response-envelope.json`](https://genai-security-project.github.io/agent-control-standard/schema/v0.1.0/response-envelope.json) `$defs/JsonRpcError`

**Notes:** An SDK that branches on a refusal has three accounts of the error object, and they disagree.

* The prose puts a human-readable `message` inside `data`. The schema requires `message` on the error object, beside `code`, which is where JSON-RPC 2.0 puts it. The table names neither field.
* The prose says `data`, when present, SHOULD carry a machine-readable `reason`. The table's machine-readable identifier is the Name column (`SESSION_REFUSED` and the rest). Only the `-32000` recovery tells the agent to read `data.reason`. The schema defines neither `reason` nor those names. `code` is an unconstrained integer.
* The table's recovery reads per-code fields the schema does not define: `data.supported_versions` on `-32001`, `data.method` on `-32003`, and `data.skew_window_ms` on `-32006`. `data` is an empty schema, so any JSON value validates, and none of those fields is required. The prose calls `data` an object.
* The `-32007` row says a deny exposes `reason_code: "chain_mismatch"`. §8.6 and the schema use `reason_codes`, an array. The schema lists `chain_mismatch` as one element of that array.

The prose also says every mandated refusal maps to a fixed code. The same `-32007` row then allows a decision instead of an error. Pick one error object and write it in all three places. Keep the JSON-RPC `message` on the error object. Give `data` a schema for the fields the recovery column reads. Use `reason_codes` for the deny form of a chain mismatch. The decision-failure note already covers the recovery column's silence on whether the session continues. No open issue asks for these three surfaces to agree. Related, and about the reference Guardian's error behavior: [#163](https://github.com/GenAI-Security-Project/agent-control-standard/issues/163) and [#130](https://github.com/GenAI-Security-Project/agent-control-standard/issues/130).

---

## (Title of next comment)

**Where:**

**Notes:**
