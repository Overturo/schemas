<!-- audit-public-language: allow — a verification specification must name
     its signature algorithm and encoding precisely; a euphemism here would
     make records unverifiable. -->
# Verification — signed records under `overturo-cr/2.9`

How a third party checks a signed record offline: the envelope, the
canonical bytes, where the keys are published, and worked examples. The
schema this applies to is `https://overturo.com/schemas/receipt/v2/record.json`.

Every record exports in three flavors — all projections of the one
canonical record:

| Flavor | What it is | Integrity claim |
|---|---|---|
| `canonical` | The `overturo-cr/2` document (default; unchanged) | Platform-internal (the audit trail) |
| `dpv` | JSON-LD over the W3C Data Privacy Vocabulary (ISO/IEC TS 27560:2023 Annex A.2) | None — a tooling view of the render |
| `signed` | The **frozen snapshot** + a detached Ed25519 signature | Offline-verifiable by anyone against published keys |

**Where:** the receipt page's download buttons, and
`GET /api/v1/consent_receipts/:id?flavor=canonical|dpv|signed`
(explicit parameter; no content negotiation). An application's
`/.well-known/transparency/:application_id` record states the available
flavors and the key-discovery URL.

Only receipts with a frozen snapshot are signable. Reconstructed renders
(transactions predating snapshot records) refuse with
`422 {"error": "not_signable"}` — signing a reconstruction would launder
its weaker fidelity.

## The signed envelope

```json
{
  "receipt": { "…the frozen snapshot, verbatim…": "…" },
  "signature": {
    "algorithm": "Ed25519",
    "canonicalization": "overturo-jcs-1",
    "region": "eu",
    "key_version": "eu-1",
    "value": "BASE64_DETACHED_SIGNATURE",
    "public_key": "BASE64_PUBLIC_KEY (a convenience copy)",
    "key_discovery": "https://REGION-HOST/.well-known/witness-configuration",
    "signed_at": "2026-07-31T12:00:00Z"
  }
}
```

The signature is computed over the canonical bytes of the `receipt`
member alone. `receipt` is whatever the platform froze at fulfillment —
current receipts are the sectioned `overturo-cr/2` shape; receipts frozen
before the sectioned schema sign in their original flat shape.

## Canonicalization: `overturo-jcs-1`

The signed message is the UTF-8 JSON serialization of the `receipt`
member with:

1. **Object keys deep-sorted** lexicographically (bytewise, at every
   nesting level).
2. **Array order preserved** (order is semantically meaningful).
3. **No insignificant whitespace** (`,` and `:` separators, nothing else).
4. **Non-ASCII characters emitted raw** — never `\uXXXX`-escaped.
5. **Values pass through unchanged, and number LEXEMES are
   significant.** Authorization records can carry non-integer numbers
   (operator-supplied authority bounds pass through — e.g. a
   `"threshold": 5.0`), serialized as the platform serializes them:
   `5.0` stays `5.0`.

> **Python users:** the defaults of `json.dumps` (ASCII-escaping, spaced
> separators) produce different bytes and a false verification failure.
> Use `json.dumps(receipt, ensure_ascii=False, separators=(",", ":"),
> sort_keys=True)` — `sort_keys` sorts recursively, matching rule 1.
> Python's parse→dump round-trips number lexemes the same way the
> platform's own serializer does, so parsing the response body is safe.

> **JavaScript users:** `JSON.parse` collapses number lexemes
> (`5.0` → `5`), and the loss is undetectable after parsing — a
> re-serialized envelope produces different bytes and a false
> verification failure on any record carrying a non-integer number.
> Canonicalize from the **raw response text** with a
> number-lexeme-preserving parse; `@overturo/verify`'s `verifyRecord`
> does this when you pass it the raw string, which is why that is its
> documented input.

## Key discovery

`GET https://REGION-HOST/.well-known/witness-configuration` publishes:

- `public_key` + `key_version` — the region's current signing key,
- `keys` — every published version for the region:
  `[{"key_version": "eu-1", "public_key": "BASE64"}, …]`.

`key_version` uses the `REGION-N` convention (`eu-1`); keys are per-region
and historical versions remain published after rotation, so a receipt
signed before a rotation verifies from the same URL. The envelope's
embedded `public_key` is a convenience copy: **verifying against it alone
proves only internal consistency** — confirm it matches the published key
for the stamped `key_version` to prove issuance.

## Worked verification (Ruby)

This is the platform's own verifier, verbatim:

```ruby
require "ed25519"
require "json"
require "base64"

def canonicalize(value)
  case value
  when Hash  then value.map { |k, v| [k.to_s, canonicalize(v)] }.sort_by(&:first).to_h
  when Array then value.map { |v| canonicalize(v) }
  else value
  end
end

envelope  = JSON.parse(File.read("consent-receipt-…signed.json"))
signature = envelope.fetch("signature")
message   = JSON.generate(canonicalize(envelope.fetch("receipt")))

Ed25519::VerifyKey
  .new(Base64.strict_decode64(signature.fetch("public_key")))   # then confirm vs published key!
  .verify(Base64.strict_decode64(signature.fetch("value")), message)
# => true, or raises Ed25519::VerifyError on any modification
```

## Worked verification (Python)

```python
import json, base64
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey

envelope  = json.load(open("consent-receipt-….signed.json"))
sig       = envelope["signature"]
message   = json.dumps(envelope["receipt"], ensure_ascii=False,
                       separators=(",", ":"), sort_keys=True).encode("utf-8")

Ed25519PublicKey.from_public_bytes(base64.b64decode(sig["public_key"])) \
    .verify(base64.b64decode(sig["value"]), message)
# raises InvalidSignature on any modification
```

## Reference implementations

The worked recipes above stay the third-party path — but two first-party
library calls implement the full contract (envelope shape checks, the
published-key trust ladder, typed refusals):

- **TypeScript:** `@overturo/verify` — `verifyRecord(rawText, {publishedKeys})`
  or `{fetchKeyDiscovery: true}`; `verifyManifestPin` for the pin recipe
  below. Pass the raw response text (see the JavaScript note under
  Canonicalization).
- **Python:** `overturo` — `verify_record(envelope, published_keys=...)`;
  `verify_manifest_pin` / `manifest_content_hash`.
- **Ruby** stays recipe-plus-corpus by decision: the `overturo` gem is
  dependency-free, and the platform's own verifier is the Ruby leg of the
  conformance corpus.

All three legs iterate one fixture set — `signed_records/` in
https://github.com/overturo/conformance, generated from the real exporter — so the
recipes, the libraries, and the server cannot drift apart silently.

## Verifying a capability manifest pin (authorization records)

An authorization record's `extensions.capability_manifest` anchors the
agent declaration the delegation was made against: a versioned reference
and a content hash. The declaration content behind any version is public
— fetch it from the manifests route under the agent's transparency
record and recompute:

```ruby
require "digest"
require "json"
require "net/http"

pin      = receipt.dig("extensions", "capability_manifest")
agent_id, version = pin.fetch("reference").split("/manifest/")
manifest = JSON.parse(Net::HTTP.get(URI(
  "https://REGION-HOST/.well-known/transparency/agents/#{agent_id}/manifests/#{version}"
)))

recomputed = "sha256:" + Digest::SHA256.hexdigest(
  JSON.generate(canonicalize(manifest.fetch("content")))   # same canonicalize as above
)
recomputed == pin.fetch("hash")  # => true: the published declaration is the one pinned
```

The hash anchors what the operator **declared** (provenance is labeled
`declared`), not that the declaration is true. A render whose
`receipt_metadata` carries `manifest_superseded: true` means the agent's
live declaration has drifted from the pinned version — the pin itself
never changes.

## Authorization records: flavors, the runtime join, the dispute kit

Authorization records (`record_type: "authorization_record"`) export in
the same three flavors from `GET /api/v1/authorization_receipts/:id`
and the record page's download group:

- **canonical** — the rendered document (current lifecycle + status).
- **signed** — the frozen snapshot + detached signature, verified
  exactly as above (same envelope, same canonicalization, same key
  discovery). Reconstructed renders refuse (`not_signable`).
- **dpv** — **only where the record carries a personal-data slice**
  (`pii_processing` present: the grant declared processing purposes).
  Pure authority semantics — bounds, escalation, exercise — have no DPV
  vocabulary, and the export refuses (`dpv_unavailable`) rather than
  emitting pseudo-DPV terms. `ovt:recordStatus` carries the raw record
  status for every kind; authority statuses (`suspended`, `pending`)
  have no `dpv:hasConsentStatus` concept and appear only there.

### The runtime↔durable join

Every honored runtime receipt (the short-lived authorization JWT) and
the durable record are two faces of one authorization, joined by
references both already carry: the JWT's claims include `grant_id` and
`chronicle_id`; the durable record's `record.record_id` is the same
grant reference, and the dispute kit's `runtime_join.consumptions`
lists each issued receipt's `jti`, `chronicle_id`, `issued_at`,
`decision`, and `context_hash`. To check a receipt you honored months
ago: find your `jti` in the kit's consumption slice and confirm the
`grant_id` claim matches the signed record's `record_id`. Completion
credentials repeat the same pair (`original_receipt_jti`,
`chronicle_id`).

Two boundary facts, stated plainly. The runtime receipt's TTL bounds
**honoring**, not evidence — an expired receipt still verifies
cryptographically forever, which is exactly what a dispute months later
needs. And a resolved join proves the two artifacts describe the
**same authorization** — linkage, never an endorsement of the action's
merits.

### The dispute kit

`overturo:dispute-kit:v1` — one download from the record page for the
record holders (principal and operator): the **signed record** (the
integrity claim), the **lifecycle** timeline (render-derived, labeled as
such), the **runtime_join** slice (bounded; a `truncated` flag appears
when capped), the **oversight_package** member (the full evidence package) exactly when a human
decision is in scope, and this guide + key discovery under
`verification`. Everything in the kit is assembled from the same
exports the flavor endpoints serve — verify the `signed_record` member
with the worked examples above.

## Signed receipt vs evidence package

- The **signed receipt** is the *subject-portable* artifact: one
  document, one signature, offline verification.
- **Evidence packages** are the *organizational* anchored-proof artifact:
  anchored audit history for compliance workflows.

They reference the same canonical structure; use the one matching who is
asking.

## The DPV flavor

The `dpv` flavor maps the rendered document onto the W3C Data Privacy
Vocabulary (ISO/IEC TS 27560:2023 Annex A.2) with a local `ovt:` context for
platform facts the vocabulary cannot express. The mapping is the product's
export vocabulary and is outside this schema's governance; the `ovt:` context
IRI is a namespace identifier and is not dereferenceable by design.
