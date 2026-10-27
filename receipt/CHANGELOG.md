# Changelog — the Overturo record schema (`overturo-cr`)

Newest first. Each entry names the version, the date, the class of change
(editorial, additive, breaking), what changed, the schema's digest after the
change, and the proposal it answers where there is one. The product's build
holds the head entry's digest equal to the served file. Entries before
2026-09-12 are reconstructed from the product's history and carry no digest;
governance starts with the entry above them. Versions 2.1 and 2.2 were never
published.

## overturo-cr/2.14 — 2026-10-24 (additive)

Compatibility: additive — one `event_channel` value added; nothing removed,
renamed or re-typed.

- `event[].event_channel` gains `assistant`: the person made the withdrawal or
  objection through a connected assistant (an AI client acting for them with
  their consent). It is the person's own act, recorded as such;
  the assistant never decides.

Schema digest: sha256:0b57395421a8bd1469d1c212870f4fdb0e0dd9fa12c6aec1b824e6f62deab85c

## overturo-cr/2.13 — 2026-10-15 (additive)

Compatibility: additive — one optional member and one `event_state` value
added; nothing removed, renamed or re-typed.

- `event[].event_channel` (optional, closed enum) names where a withdrawal or
  objection was made from: the person's records (`records_page`), the
  management link in their email (`management_link`), the record link in their
  receipt (`record_link`), the organisation's mailing tool (`mailing_tool`), a
  newer record from the organisation's form (`re_attestation`), the
  organisation (`organisation`), its systems (`api`), the end of the term
  (`expiry`), the platform's subject sweep (`sweep`), or `unspecified` for a
  change whose evidence predates the channel. Present only on appended
  withdrawal and objection events; absent on the frozen prefix (206 D7).
- `event[].event_state` gains `objection recorded`: the right to object
  exercised after the decision on a legitimate-interest purpose, scoped by
  `purpose_id` (206 D2).

Schema digest: sha256:1fd166eb725083d19edda9f77d6a36c40b59e64c9fad9d524c05ec00bc3da03b

## overturo-cr/2.12 — 2026-10-15 (editorial)

Compatibility: editorial — no member, enum or rule changed.

- The published interpretation's claim boundary names the control Overturo
  renders inside an organisation's form by its public name, "the Overturo
  field"; the `collection_channel` values `embedded_field` and
  `embedded_field_browser` are unchanged on the wire.

Schema digest: sha256:e871b444a0f3cf1222840618bc860a4523c851fd4ac9ff50e6185c99d35f7cfb

## overturo-cr/2.12 — 2026-10-12 (additive)

Compatibility: additive.

- `extensions.collection_channel`: two values, `embedded_field` and
  `embedded_field_browser`, for a control Overturo rendered inside the
  organisation's own form (committed by its server, or by the browser); the
  description now states which channels are Overturo-rendered.
- `extensions.rendering_evidence`: present only for those channels — who
  rendered the control, the notice version shown, when, the host page's origin
  as the browser asserted it, whether the person changed the control with a
  trusted gesture, and the control's visibility at submit with the kind of
  measurement the browser offered.

Schema digest: sha256:e871b444a0f3cf1222840618bc860a4523c851fd4ac9ff50e6185c99d35f7cfb

## overturo-cr/2.11 — 2026-10-12 (additive)

Compatibility: additive.

- `extensions.captured_at`, `extensions.recorded_at`: the organisation's capture
  time beside the platform's recording time for a captured record (an import
  keeps its original capture time; the duration counts from it).
- `extensions.presentation_attested`: the purposes whose optional presentation
  the organisation attested rather than the platform read.
- `extensions.wording_drift.purposes`: both digests and the verdict per purpose
  when the wording was attested purpose by purpose.

Schema digest: sha256:d807e4e3f76f8fbefb8d0b5835663feebfe0d2b6a70f7b5c1e98b0fe86f2fb17

## overturo-cr/2.10 — 2026-10-12 (additive)

Compatibility: additive.

- `extensions.collection_channel`, `extensions.collection_platform`: where and
  by whom a decision was captured, with the attestation boundary stated in the
  description.
- `extensions.wording_drift`: the attested-versus-published wording verdict for
  decisions captured outside Overturo's surfaces.

Schema digest: sha256:72821ef97a8ad39df42374684b206455b8b14f9fd31c2aa4264007e73b3c7bd5

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
