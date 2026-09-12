# The Overturo record schema (`overturo-cr/2.9`)

This directory publishes the JSON Schema for consent, notice and authorization
records — the receipts the Overturo platform issues — together with its
interpretation and its verification guide.

- **Identifier:** `https://overturo.com/schemas/receipt/v2/record.json` — the schema's `$id`.
- **How a record finds it:** a record names its version in
  `record.schema_version` (`overturo-cr/MAJOR.MINOR`); the schema for major N
  is served at `https://overturo.com/schemas/receipt/vN/record.json`.
- **Conforms to:** ISO/IEC TS 27560:2023.
- **This copy:** `v2/record.json`, byte-equal to the file served at the
  identifier (digest `sha256:886fb8e380c73a7d5499b13a034b1c170350d5f65c78f4529afb495545c7df5a`). The product's build holds the two equal;
  the served file and this file are the same bytes.
- **History:** [CHANGELOG.md](./CHANGELOG.md) — every version since 2.0 and
  every state the served file has been in since this repository governs it.
- **Interpretation:** [INTERPRETATION.md](./INTERPRETATION.md) — record kinds,
  the standard's section map, the extension members, conformance language.
- **Verification:** [VERIFICATION.md](./VERIFICATION.md) — the signed envelope,
  canonicalization, key discovery, worked examples.

## Governance

The Overturo Geneva Association governs this schema: its versioning policy,
its change process and its interpretation. The operating companies (OmVi Labs
Inc and Overturo Ltd) issue records under it and implement it in the product.

## Serving commitment

The identifier's host is operated by the companies. They serve, at every
identifier under `/schemas/receipt/`, exactly the text published here. A
superseded major stays served unchanged at its own identifier for
five years after its successor is published.
Served majors: `v2`.

## Issuing records under this schema

Anyone may issue records that reference this schema. A record names the
version it follows in `record.schema_version`; a verifier resolves the
major's identifier and validates against these bytes. Conformance claims about a
product are the association's to define and are not made by this repository.

## Name

`overturo-cr` and the version string are identifiers, not a trademark grant.
The mark stays with its owners; renaming the identifier is a major version.

## Versioning policy

Versions read `overturo-cr/MAJOR.MINOR`. Three classes of change:

- **editorial** — a description changes; no member, enum or rule changes.
  The version stays; the changelog records the new digest.
- **additive** — an optional member or an enum value is added; nothing is
  removed, renamed, re-typed or tightened. The minor version rises.
- **breaking** — anything else. A new major at a new identifier
  (`…/v3/record.json`) with its own file; the previous major's file
  stays served unchanged at its own identifier for five years.

## Changing the schema

A change lands in the product only with a changelog entry: the version, the
date, the class of change, what changed, the schema's digest after the change,
and the proposal it answers where there is one. The product's build refuses a
schema change without its entry, and holds the served file, this copy, the
version the product stamps into records and the changelog head together.

Proposals arrive as issues or pull requests here: https://github.com/overturo/schemas/issues. Accepted
changes are ported into the product and appear in the next release snapshot.
`bin/receipt-schema-diff PREVIOUS.json PROPOSED.json` (in the product
repository) reports whether a change is additive.

## What conformance means here

ISO/IEC TS 27560 is a personal-data-consent standard: conformance language applies to records that carry personal-data processing.
A pure authority record follows the standard's structure and is 27560-aligned, never more.

## Related

- Golden records for every kind, signed: `signed_records/` in https://github.com/overturo/conformance.
- The open verifier: https://github.com/overturo/verify (`verifyRecord`).
- The public documentation page: https://overturo.com/developers/docs?tab=receipts.
- Paste a signed record and watch it verify in the browser: https://overturo.com/verify.

## Licence

Apache-2.0 — copyright Overturo Geneva Association. See the repository's
`LICENSE`.
