# Core Governance Declaration

**Spec:** `core-governance` · v0.1 · Kinetic Gain
**Status:** Draft — open spec, implementation in progress
**Schema:** [`schema.json`](./schema.json) (JSON Schema 2020-12) · **Example:** [`example.json`](./example.json)

## What this is

A machine-readable declaration of what an AI model or agent can do, how it is classified under recognized risk frameworks, what data it touches, and where its evidence lives. One JSON document a buyer's compliance team can read, validate against the schema, and verify — instead of a PDF model card and a leap of faith.

This is the governance and evidence **declaration layer**. It is composable with, not a competitor to, the protocols around it. MCP carries how an agent connects to tools and data. CycloneDX ML-BOM carries component provenance. This spec declares capability, classification, posture, and evidence — and references the others where they fit (`subject.provenance.cyclonedx_ref` links a BOM; an MCP server can advertise its governance declaration).

A declaration is an **assertion backed by evidence, not a certification.** Kinetic Gain does not certify or attest compliance. The `regulatory_class` block records who asserts a classification and when. The `evidence` block points to the hash-chained audit stream and the attestation endpoint a verifier checks independently. The reader trusts the evidence, not the adjective.

## The shape

| Field | Required | Purpose |
|---|---|---|
| `kg_version` | yes | Pins the document to a schema version (`"0.1"`). |
| `subject` | yes | What is governed — `id`, `type` (model/agent/pipeline/tool), `name`, `platform`, `version`, optional `provenance` (incl. a CycloneDX BOM link). |
| `capabilities[]` | yes | Declared actions, each with a `data_scopes` list and an `autonomy_level` (assistive / supervised / autonomous). At least one. |
| `regulatory_class` | yes | A self-asserted risk class against a named `framework` (`eu_ai_act` / `nist_ai_rmf` / `kg_default`), with `rationale`, `asserted_by`, `asserted_at`. |
| `data_posture` | yes | `access` (read_only / scoped_write), data `classifications`, `residency`. |
| `evidence` | yes | `audit_stream` (hash-chained events), `attestation` (a verifier endpoint), `method`. |
| `status` | yes | `live` / `mapped` / `pending` — honest deployment state. |

## Honesty is machine-checkable

The schema encodes one rule as an `if/then` conditional: a declaration with `status: "live"` **must** carry an `evidence.attestation` URI. You cannot declare a control live without pointing to something a verifier can fetch. `mapped` and `pending` declarations carry no such requirement — they are honest about not being shipped yet.

This is the same anti-vapor rule that governs the public proof surfaces (`watsonx.kineticgain.com`, `genesys.kineticgain.com`), pushed down into the data model. A validator that rejects a `live` declaration with no attestation is the honesty policy made executable.

## How it backs the estate

The `watsonx` and `genesys` proof surfaces cite this spec for their capability and regulatory-class proof points. Until this repo existed, those points had no backing artifact and were dropped under the anti-vapor rule. With the spec live, they rebind here, and each can be promoted from `mapped` toward `live` as the attestation endpoint ships.

## Validating a declaration

```bash
# any JSON Schema 2020-12 validator works; example with ajv
npx ajv-cli validate -s schema.json -d example.json --spec=draft2020
```

## Versioning

`v0.1` is a draft. Breaking changes bump the minor version until `v1.0`; the `kg_version` field pins each document to its schema. Drafts may change; pin to a version in production.

## Not a certification

This spec defines a declaration format. Publishing a declaration asserts a position and points to evidence. It does not assert certification, accreditation, or regulatory compliance, and Kinetic Gain makes no such claim on a publisher's behalf.
