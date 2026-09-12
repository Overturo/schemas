> **Release mirror.** This repository is a read-only snapshot of
> `schemas` 1.1.0, published from Overturo's main
> development repository. Issues and pull requests are welcome here; accepted
> changes are ported upstream and appear in the next release snapshot.
> Security reports: see [SECURITY.md](./SECURITY.md).

# Overturo shared SDK schemas

Canonical JSON Schemas consumed by every Overturo SDK that handles authorization
decisions, whether it requests them or oversees them.

| Schema | Version | Shape |
|---|---|---|
| `overturo_decision_v1.0.0.json` | 1.0.0 | `OverturoDecision` — one authorization decision |
| `overturo_block_invocation_v1.0.0.json` | 1.0.0 | `BlockInvocation` — one step of a decomposed decision |
| `receipt/v2/record.json` | see [`receipt/README.md`](./receipt/README.md) | The consent, notice and authorization record — governed by the association |

## One shape

These schemas are the single source of truth: an SDK that oversees decisions
consumes `OverturoDecision` rather than defining a parallel shape, and an
oversight-specific example lives in the shared conformance corpus
(`fixtures/` in https://github.com/overturo/conformance, `mode: "oversight"`).

## Versioning

Schema files carry the version in the filename. Backward-compatible
additions are allowed within a major (`1.x` → `1.x+1`). Breaking changes
require a new file (`overturo_decision_v2.0.0.json`); old version stays
for compatibility window.

Wire response bodies carry the active version in
`overturo_decision_schema_version` so SDKs can detect drift.

## Governance

The record schema under `receipt/` follows its own policy, stated in
[`receipt/README.md`](./receipt/README.md): the Overturo Geneva Association
governs it, the operating companies issue records under it, and its changelog
is the record of every version. The two decision schemas above keep the
versioning note in this file.

## Licence

Apache-2.0 — copyright Overturo Geneva Association. See `LICENSE`.
