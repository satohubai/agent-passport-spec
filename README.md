<div align="center">

# ⬡ Sato Agent Passport

**A machine-readable identity + verification manifest for onchain agents.**

`sato.agent.manifest/v1`

**[Spec](SPEC.md)** · **[Register an agent](https://satohub.ai/agents/register?utm_source=github&utm_medium=spec-repo&utm_campaign=passport)** · **[Live registry](https://satohub.ai/agents?utm_source=github&utm_medium=spec-repo&utm_campaign=passport)**

</div>

---

## The problem

Onchain agents transact with strangers. Today, "who is this agent, who controls its wallet, and does its track record check out?" has no standard answer — every marketplace invents its own profile page, and none of it is machine-readable.

## The passport

Every registered agent gets:

- a **public profile** at `satohub.ai/agents/<slug>`
- a **versioned JSON manifest** at `GET https://satohub.ai/api/agents/<slug>/manifest` — schema `sato.agent.manifest/v1` ([full spec](SPEC.md))
- **wallet verification by signature** — the agent (or its builder) signs a challenge with the agent's wallet key (EIP-191); passports issue instantly on proof of key control
- **on-chain verification checks**, each by its own method: ERC-8004 registration, MCP/A2A endpoint probes, x402 discovery, multi-chain wallet activity

## The honesty rules (load-bearing)

1. **Wallet verification proves the key, never the product.** A signed challenge means someone controls that wallet — nothing more.
2. **Everything else is self-reported until independently reviewed.** The manifest says so, in-band, in `trust.note` and `identity.note`.
3. **Payment metadata is published as metadata, never endorsement.**
4. Checks that can't be proven return `null` — the manifest never fakes a field.

## Quick look

```json
{
  "schema": "sato.agent.manifest/v1",
  "sato_agent_id": "sato:agt:xxxxxxxxxxxx",
  "name": "…",
  "chains_supported": ["Base"],
  "stack": [{ "slug": "elizaos", "resource_url": "https://satohub.ai/resources/elizaos" }],
  "identity": {
    "wallet_address": "0x…",
    "wallet_verified": true,
    "wallet_verification_method": "eip191-signature",
    "standards": ["x402", "erc-8004"]
  },
  "trust": { "verification_status": "Self-Reported", "liveness_status": "Active" },
  "onchain": { "erc8004": "…", "mcp": "…", "a2a": "…", "x402": "…" },
  "registry": { "registrar": "Sato Hub", "manifest_url": "…" }
}
```

Full field reference: **[SPEC.md](SPEC.md)**.

## Status

v1 is live and serving manifests for every agent in the [registry](https://satohub.ai/agents?utm_source=github&utm_medium=spec-repo&utm_campaign=passport). The spec is versioned; breaking changes bump the schema string. Feedback and proposals → issues on this repo.

---

<sub>Maintained by [Sato Hub](https://satohub.ai?utm_source=github&utm_medium=spec-repo&utm_campaign=passport). Spec text CC-BY-4.0. © 2026 Prime Signal LLC.</sub>
