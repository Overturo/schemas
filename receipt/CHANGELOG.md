# Changelog — the Overturo record schema (`overturo-cr`)

Newest first. Each entry names the version, the date, the class of change
(editorial, additive, breaking), what changed, the schema's digest after the
change, and the proposal it answers where there is one. The product's build
holds the head entry's digest equal to the served file. Entries before
2026-09-12 are reconstructed from the product's history and carry no digest;
governance starts with the entry above them. Versions 2.1 and 2.2 were never
published.

## overturo-cr/2.9 — 2026-09-12 (editorial)

Compatibility: editorial — no member, enum or rule changed.

- Member descriptions reworded to describe behaviour only: internal programme
  references removed; standard citations read "ISO/IEC TS 27560 clause N".
- First governed publication: this changelog, the README, the interpretation
  and the verification guide.

Schema digest: sha256:886fb8e380c73a7d5499b13a034b1c170350d5f65c78f4529afb495545c7df5a

## overturo-cr/2.9 — 2026-08-14 (additive)

Compatibility: additive.

- `extensions.signature_evidence`: role-verification and qualified-signature
  evidence ride as pointer plus digest; values never inlined.

## overturo-cr/2.8 — 2026-08-04 (additive)

Compatibility: additive.

- `extensions.agent_disclosures` on consent and notice records: the AI agents
  disclosed to the human, one entry per agent, with the disclosure's
  provenance (`platform` or `declared`) and the capability-manifest version
  the disclosure card rendered.

## overturo-cr/2.7 — 2026-08-04 (additive)

Compatibility: additive.

- `extensions.capability_manifest` on authorization records: the
  content-addressed agent declaration the grant was made against (reference
  plus hash), pinned at freeze; supersession is a render-time overlay.

## overturo-cr/2.6 — 2026-08-04 (additive)

Compatibility: additive.

- Authority lifecycle event states on authorization records: the frozen prefix
  carries `authority requested` and `authority granted`; renders append
  suspension, revocation, expiry and exercise from recorded evidence.

## overturo-cr/2.5 — 2026-08-04 (additive)

Compatibility: additive.

- `record_type: authorization_record` — one frozen record per delegated-authority
  commitment; the `authority` section states the enforced grant verbatim.
- `party_identification` rows for authorization records: `principal` and
  `agent` roles; `software_agent` as a party kind; the agent accountability
  record reference.

## overturo-cr/2.4 — 2026-07-31 (additive)

Compatibility: additive.

- `record_type: notice_record` — notice-not-consent mechanisms and anonymous
  completions; versioned notice references (`privacy_notice`) with a content
  fingerprint.
- Every member documented: no undescribed field on the published contract.

## overturo-cr/2.3 — 2026-07-30 (additive)

Compatibility: additive.

- The `event` section: the lifecycle as a projection of recorded evidence,
  with the closed `event_state` vocabulary.

## overturo-cr/2.0 — 2026-07-30 (breaking)

Compatibility: breaking — the first sectioned record (a new major).

- The record organized into the ISO/IEC TS 27560:2023 sections (`record`,
  `pii_processing`, `party_identification`, `event`, `extensions`); the
  controller identity record; purpose and processing facts (retention,
  storage, recipients, withdrawal); the documented extension members.
