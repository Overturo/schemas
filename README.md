> **Release mirror.** This repository is a read-only snapshot of
> `schemas` 1.0.0, published from Overturo's main
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

## One shape

These schemas are the single source of truth: an SDK that oversees decisions
consumes `OverturoDecision` rather than defining a parallel shape, and an
oversight-specific example lives in the shared conformance corpus
(`lib/sdk/shared/conformance/fixtures/`, `mode: "oversight"`).

## Versioning

Schema files carry the version in the filename. Backward-compatible
additions are allowed within a major (`1.x` → `1.x+1`). Breaking changes
require a new file (`overturo_decision_v2.0.0.json`); old version stays
for compatibility window.

Wire response bodies carry the active version in
`overturo_decision_schema_version` so SDKs can detect drift.
