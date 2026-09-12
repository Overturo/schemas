# Interpretation — `overturo-cr/2.9`

What the members of a record mean, which record kind carries what, and what
conformance language applies. The member descriptions below are the schema's
own text; this document adds only what no schema field holds.

## Record kinds

- **`consent_record`** — Consent records carry the granted purposes and the consent lifecycle. *Conformance language applies: the record carries personal-data processing.*
- **`notice_record`** — Notice records cover notice-not-consent mechanisms and anonymous completions (no principal identifier — the record identifier stands in). *Conformance language applies to the notice it records; a notice event is never rendered as consent given.*
- **`authorization_record`** — Authorization records are durable evidence of a delegated-authority commitment. Every commitment freezes exactly one record at activation; lifecycle history is derived at render time and never rewrites the frozen record. A held runtime receipt resolves into the same record through the references both already carry (the dispute kit's join slice — see the verification guide), and every agent-side fact is provenance-labeled: platform-attested registration facts versus operator-declared metadata, never blurred. *Aligned only, unless the record carries a personal-data slice (a pii_processing section): pure authority semantics are outside the standard's subject matter.*

Absence is meaningful throughout: a field the platform cannot state truthfully is omitted, never guessed — each field's absence semantics are documented in the schema itself.

## Conformance language

ISO/IEC TS 27560 is a personal-data-consent standard: conformance language applies to records that carry personal-data processing.
A pure authority record follows the standard's structure and is 27560-aligned, never more.

## Alignment

- **ISO/IEC TS 27560:2023** — The record is organized into the standard's sections; the extensions member carries facts the standard has no slot for, each documented in the schema itself.
- **Kantara Consent Receipt v1.1** — The extensions member follows the extension-slot rule: nothing standard-named carries non-standard semantics.
- **Kantara ANCR (Stage 1)** — Notice records are Kantara ANCR Stage-1 aligned: a notice event is never rendered as consent given.

## The sections

Top-level members, in schema order, with the schema's description of each and
the ISO/IEC TS 27560:2023 clauses their member descriptions cite.

| Member | Clauses cited | Description |
|---|---|---|
| `record` | none cited | The record's identity: the schema version it follows, the standard it conforms to, the record identifier, the kind, and whether the record is anonymous. |
| `pii_processing` | 6.3.4.2, 6.3.4.5, 6.3.4.7, 6.3.4.9, 6.3.4.10 | The granted purposes with their decision-time facts: language and jurisdiction, the notice as presented, the collection method, each purpose's lawful basis, categories, retention, storage and recipients, and how consent is withdrawn. |
| `party_identification` | 6.3.6 | The controller (declared legal identity when the org completed its controller identity record — provenance 'declared'; display-name fallback otherwise), the platform processor row, and the catalogued supervisory authority. Identity data is DECLARED by the organization, never verified by the platform. party_role values in use: controller, processor, supervisory_authority (string-typed — later members may add roles). |
| `authority` | none cited | Authorization records only. States the enforced grant verbatim: each bound dimension appears exactly when the platform enforces it - absence means not configured, never 'unlimited'. Bound-dimension inner shapes are owned by the platform's authority-bounds vocabulary; this schema constrains presence, not structure. |
| `event` | none cited | The lifecycle as a projection of the platform's existing evidence (tamper-evident audit-trail records, transaction timestamps) — never a new event store. Consent records freeze the pre-decision prefix (notice shown, consent requested) plus the given event, in canonical lifecycle order; notice records freeze notice vocabulary instead — see event_state. Renders append post-decision transitions (withdrawn — optionally scoped by purpose_id — and expired) derived at read time. Arrays are heterogeneous by construction: flows whose delivery path does not record an opening start at "consent requested"; records issued before the lifecycle section existed start at "consent given"; system-mediated records are notice records carrying the single constructive "notice shown". "consent refused" is defined but unissued in v1 (records exist only for completed decisions; refusal evidence belongs to the notice-receipt track). States "consent invalidated" and "consent halted" remain reserved: no platform transition claims them yet. Actor attribution: the principal for given/withdrawn/acknowledged (anonymous records substitute record_id); the controller (application) for notice shown/requested; the platform processor for expiry. Authorization records: the frozen prefix carries 'authority requested' and 'authority granted'; renders append the lifecycle derived from existing evidence (the grant's chronicle trail, the receipt-consumption aggregate) — 'authority exercised' rows are AGGREGATES (see occurrences), suspension/resumption pairs are dated history, approvals carry the human decision with real attribution, and 'authority expired' is suppressed after a revocation. Records frozen before lifecycle events were recorded start at 'authority granted' (frozen means frozen). Histories without audit-trail records emit only what their columns can date. |
| `extensions` | 6.3.2.2 | Documented platform extensions (Kantara CR v1.1 clause 4.2; ISO/IEC TS 27560 clause 6.3.2.2): facts the standard has no slot for. Every key is documented here; nothing standard-named carries non-standard semantics. |

## Extension members

Facts the standard has no slot for. Every key is documented; nothing
standard-named carries non-standard semantics.

- **`collection_mechanism`** — The raw platform consent-mechanism value; pii_processing.collection_method carries the human label.
- **`delivery_mode`** — How the decision surface was delivered: redirect, embed, popup, or link.
- **`scopes`** — The effective granted platform permissions at fulfillment — operational grant state, distinct from the purpose grants above.
- **`sensitive`** — True when the flow collected at least one field the organization classified at a sensitive level.
- **`consent_flow_id`** — The stable public identifier of the flow (the versioned notice reference is rooted in it).
- **`grant_integrity`** — Authorization records only: the reference to the grant's dual signature - content hash + key versions. The signatures themselves are the signed-export payload, never inlined here.
- **`delegation_chain_root`** — Authorization records only: the root commitment's record reference when this authority was sub-delegated. Absent for root grants.
- **`capability_manifest`** — Authorization records only: the capability-manifest version the grant was made against — the agent's declaration, content-addressed. The hashed content set is exactly {agent_type, name, delegated_scopes (sorted), and the operator-declared disclosure metadata: capabilities, operator_name, privacy_policy_url, data_storage, retention_period} — canonical deep-sorted JSON, sha256; growing the set is a schema minor, never silent. agent_type/name/delegated_scopes are platform-attested registration facts; the metadata keys are operator-DECLARED (the hash anchors what was declared, not that it is true). A model/version fingerprint has no substrate field today and is documented-absent from the set. Manifest drift is AGENT-side (what the agent declares); composition drift is FLOW-side (what a flow executes) — separate mechanisms, never conflated. Absent on records frozen before manifest pinning existed and when no unique agent resolves. Pinned at freeze; supersession never rewrites it (renders overlay manifest_superseded instead).
- **`accountability_record`** — Structural slot: the agent accountability record reference in effect at activation. Documented-absent until that member ships.
- **`agent_disclosures`** — Consent and notice records only: the AI agents disclosed to the human this record belongs to — one entry per agent, EU AI Act Art. 50 aligned. A disclosure entry proves NOTICE (the human was told), never grant or authority: the commitment story lives in authorization records, and this array is schema-forbidden there. Present exactly when the flow disclosed agents (platform provenance) or the operator declared an out-of-band disclosure (declared provenance). Carries agent-side references only — never the counterparty's identity (the anonymous kind stays anonymous through these fields).
- **`signature_evidence`** — Signing-party verification evidence: one entry per completed evidence envelope anchored to the signing party, sorted by completed_at. Pointer + digest only — verdict/claim CONTENT lives in the agreement export and evidence packages, never here. Present on consent records only, and only when the signing party's transaction carries completed evidence; honest-absent otherwise.

## Event states

The lifecycle is a projection of recorded evidence — nothing is synthesized.
The frozen record carries the pre-decision events; renders append
post-decision transitions (withdrawal, expiry) derived from the same evidence
at read time.

- `notice shown`
- `notice acknowledged`
- `consent requested`
- `consent given`
- `consent refused`
- `consent withdrawn`
- `consent expired`
- `authority requested`
- `authority granted`
- `authority exercised`
- `authority suspended`
- `authority resumed`
- `authority revoked`
- `authority expired`
- `approval requested`
- `approval granted`
- `approval denied`
- `approval countered`

## Export flavors

- **`canonical`** — this schema's document (the default).
- **`dpv`** — JSON-LD over the W3C Data Privacy Vocabulary (27560 Annex A.2). Authorization records that carry no personal-data processing refuse this flavor: pure authority semantics have no DPV vocabulary.
- **`signed`** — the frozen record plus a detached signature, verifiable offline.

The verification procedure for the signed flavor is in
[VERIFICATION.md](./VERIFICATION.md).
