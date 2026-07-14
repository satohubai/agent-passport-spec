# Sato Agent Passport — Manifest Specification

**Schema:** `sato.agent.manifest/v1` · **Status:** live · **Endpoint:** `GET https://satohub.ai/api/agents/<slug>/manifest`

The manifest is a versioned JSON document describing one onchain agent: identity, stack, links, payments, verification state, and on-chain checks. It is served by the registrar (Sato Hub) and append-versioned — every accepted change adds a manifest version, so the sequence doubles as the passport's changelog.

## Top-level fields

| Field | Type | Meaning |
|---|---|---|
| `schema` | string | Always `"sato.agent.manifest/v1"` for this version |
| `sato_agent_id` | string | Non-enumerable registry id, `sato:agt:<12 base32>` |
| `name` / `slug` | string | Display name; URL slug |
| `description` | string | Creator-written, hype-screened one-liner |
| `agent_type` | string | What kind of agent (trading, service, social, …) |
| `chains_supported` | string[] | Chains the agent operates on (self-declared) |
| `model_provider` | string\|null | LLM provider, if declared |
| `framework` | string\|null | Agent framework, if declared |
| `stack` | object[] | Validated directory slugs: `{ slug, resource_url }` — the agent↔resource-graph join |
| `stack_notes` | string\|null | Free-text notes on the stack |

## `links`

`website`, `repo`, `docs`, `x`, `endpoint` — all string\|null. `endpoint` is the agent's callable service URL, if any.

## `payments`

| Field | Type | Meaning |
|---|---|---|
| `enabled` | boolean | Creator says the agent accepts payment |
| `x402` | boolean | Supports HTTP-402/x402 |
| `endpoint` | string\|null | Payment endpoint |
| `pricing_model` | string\|null | Free text (e.g. "per-call USDC") |
| `note` | string | Always present: payment metadata is self-configured, not an endorsement |

## `identity`

| Field | Type | Meaning |
|---|---|---|
| `wallet_address` | string\|null | The agent's wallet (EVM checksummed or base58) |
| `wallet_verified` | boolean | Key control proven by signature |
| `wallet_verified_at` | timestamp\|null | When |
| `wallet_verification_method` | string\|null | `"eip191-signature"` in v1 (EOA); ERC-1271 + ed25519 planned |
| `standards` | string[] | Declared standard support: `x402`, `erc-8004`, `mcp`, `a2a` |
| `note` | string | In-band honesty note: verification proves the key, not the product |

**Verification flow (v1):** `GET /api/agents/challenge?name&wallet` → canonical challenge (binds wallet + name + 15-minute timestamp) → EIP-191 `personal_sign` → `POST /api/agents/register` with `wallet_signature` + `signed_at`. Signed registrations issue instantly; unsigned ones queue for human review.

## `trust`

| Field | Type | Meaning |
|---|---|---|
| `verification_status` | string | `Self-Reported` at registration, always. `Verified`/`Audited` only via evidence-gated review — nothing in the registration pipeline can mint them |
| `liveness_status` | string\|null | Observed activity signal |
| `note` | string | In-band: self-reported unless explicitly marked otherwise |

## `onchain`

A compact projection of the registrar's verification report, or `null` until checks have run. Each check is independent and named by method — a passing check proves exactly what the method probes, nothing more:

- `erc8004` — registration in the ERC-8004 identity registry (multi-chain `balanceOf` proof + agent id)
- `mcp` — the declared MCP endpoint answered an initialize + tools/list probe
- `a2a` — `/.well-known/agent-card.json` served a valid card
- `x402` — listed in x402 discovery (Coinbase Bazaar payTo/host match)
- `activity` — wallet transaction activity (multi-chain, public sources)
- `checked_at` — when the report was produced (re-run daily by the registrar)

## `registry`

`registrar` ("Sato Hub"), `profile_url`, `manifest_url`, `claimed_resource` (directory listing this agent claims, if any), `registered_at`, `updated_at`.

## Versioning

The `schema` string is the contract. Additive fields may appear within v1; breaking changes bump to `/v2` and v1 continues to be served. Manifest versions are append-only per agent.

## Design principles

1. **Machine-first.** Agents transact with agents; the manifest is for their consumption, the profile page is the human view of the same data.
2. **Provable or null.** No field pretends. Verification distinguishes *key control* (signature), *endpoint liveness* (probes), *on-chain registration* (registry reads), and *human review* (evidence-gated) — and names which one it means.
3. **Registrar-neutral by construction.** The schema carries `registry.registrar`; nothing in the format is Sato-specific except the issuing endpoint.

---

<sub>Spec text CC-BY-4.0 · reference implementation lives at [satohub.ai](https://satohub.ai?utm_source=github&utm_medium=spec-repo&utm_campaign=passport) · © 2026 Prime Signal LLC</sub>
