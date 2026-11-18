<div align="center">

<img src="images/logo.png" alt="Tereno" width="150" />

# tereno-skills

### 🛡️ Verify before you act — Official Agent Skills for Tereno on Base

**Pre-action onchain security cache, exit simulation, cabal detection, and oracle dispute audits over x402.**
Every verdict returns structured evidence, explicit limitations, and **the exact block or timestamp it stops being valid**. Reusing another agent's fresh check costs **$0.01–$0.02**, and seeding new checks earns **20% reuse dividends**.

<br/>

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-SKILL.md-8B5CF6?style=for-the-badge)](https://agentskills.io)
[![ClawHub](https://img.shields.io/badge/ClawHub-%40tereno--xyz%2Ftereno-00C853?style=for-the-badge)](https://clawhub.ai)
[![Base](https://img.shields.io/badge/Base-0052FF?style=for-the-badge&logo=coinbase&logoColor=white)](https://base.org)
[![x402](https://img.shields.io/badge/x402%20%C2%B7%20USDC-2775CA?style=for-the-badge)](https://www.x402.org)

[![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)](./LICENSE)
[![baw compatible](https://img.shields.io/badge/Binance%20Wallet-baw%20x402-F0B90B?style=flat-square)](./skills/tereno/references/baw-x402-workflow.md)
[![free trial](https://img.shields.io/badge/MCP%20trial-1%20free%2Fday-success?style=flat-square)](#-compatible-payment-paths)

```bash
npx skills add tereno-xyz/skills
```

</div>

---

## ⚡ Why this exists

Autonomous agents executing swaps on Base or placing prediction-market bets often act on blind RPC guesses or static blocklists. Auditing bytecode, simulating exits, tracing deployer funding clusters, and checking UMA oracle clauses requires **8–12 sequential RPC calls** that blow serverless memory caps and timeouts.

`tereno-skills` teaches your agent how to use [Tereno](https://www.tereno.xyz) cleanly — whether via **Binance Agentic Wallet (`baw`)**, **HTTP x402**, or **MCP**:

- **No API key, no signup.** Your agent's Base wallet *is* the identity.
- **Deterministic policy verdicts.** Every capability returns `verdict` (`allow` | `review` | `block`), `reasonCodes`, `evidence`, `limitations`, `validUntilBlock`, `artifactId`, and a cryptographic `receipt`.
- **Cache-first & positive-sum.** Cache hits resolve in `< 200ms` at `$0.01–$0.02`. Cold misses seed the shared artifact — earning the seeder **20% reuse dividends** in credits every time another wallet reuses it.

---

## 🚀 Install

### Via `skills` CLI (Claude Code, Cursor, Gemini CLI, OpenClaw, Codex)

```bash
npx skills add tereno-xyz/skills
```

### Via OpenClaw / ClawHub

```bash
openclaw skills install tereno
```

After updating this repository, reinstall the skill from GitHub with the command above, then restart or reload skills in your agent runtime. Resolve any pending payments before restarting an agent that holds recovery state in memory. Once loaded, your agent automatically gets the full `SKILL.md` playbook plus step-by-step reference guides for `baw`, capability schemas, cooperative bounties, and `ERC-8183` / `ERC-7579` guards.

---

## 🧭 Compatible Payment Paths

This skill routes agents through three onboarding paths:

1. **Native Binance Agentic Wallet (`baw`) x402 Flow (Recommended)**
   - Trigger the `402 Payment Required` challenge, inspect Base USDC terms with `baw x402-payment preview`, sign EIP-3009 with `baw x402-payment sign`, and replay with `PAYMENT-SIGNATURE` (using `--max-time 15`).
   - Preserve the 90s price offer; after a timeout, resend the same signed request instead of signing again.
   - Full shell recipes: [references/baw-x402-workflow.md](./skills/tereno/references/baw-x402-workflow.md).
2. **Free Daily MCP Evaluation Bridge (Cold-Start Trial)**
   - Point any MCP client at `https://www.tereno.xyz/api/mcp` (or `npx -y tereno-mcp`). Every IP receives **1 free daily verification check** across paid capabilities before funding an x402 wallet.
3. **Self-Funding Cooperative Seeder Loop (`+$0.005` Bonus)**
   - Poll `GET /api/v1/bounties` for unmet demand (`misses >= 2`), compute & sign `contract.guard` bytecode fingerprints for **free** via `POST /api/v1/artifacts/publish`, earn an **instant `$0.005` credit bonus** plus **20% reuse dividends** discounting future calls down to `$0.001`.

---

## 💳 Capabilities Covered (x402 in USDC on Base)

| Capability | Method & Route | Price (Hit → Miss) | TTL | What it verifies |
|---|---|---:|---|---|
| 🚀 `token.launch.guard` | `GET /api/v1/capabilities/token.launch.guard/invoke?address=0x...` | `$0.02` – `$0.07` | `12s` | Live `eth_call` buy/sell exit simulation, DEX LP depth/lock, owner trap powers |
| 👥 `token.holders.guard` | `GET /api/v1/capabilities/token.holders.guard/invoke?address=0x...` | `$0.01` – `$0.04` | `15m` | Top-10 holder concentration & deployer-funded cabal Sybil clusters |
| 🛡️ `contract.guard` | `GET /api/v1/capabilities/contract.guard/invoke?address=0x...` | `$0.01` – `$0.03` | `10s` | Executable bytecode, EIP-1967 proxy, pause status & zero-RPC trap selectors |
| 🪙 `token.metadata` | `GET /api/v1/capabilities/token.metadata/invoke?address=0x...` | `$0.01` – `$0.03` | `10s` | Verified ERC-20 `name`, `symbol`, `decimals` (7d) & fresh `totalSupply` |
| ⛽ `chain.snapshot` | `GET /api/v1/capabilities/chain.snapshot/invoke` | `$0.01` – `$0.02` | `4s` | Latest Base block number, timestamp, base fee & suggested gas fees |
| 📊 `prediction.evidence` | `POST /api/v1/capabilities/prediction.evidence/invoke` | `$0.01` – `$0.05` | `30m` | Sourced proposition evidence, UMA dispute risk & emergency 50-50 clause audit |

> **⏱️ Latency & HTTP Timeout Guidance:** Cache hits execute in `< 200ms` (`executionMs: 0`). Cold cache misses run live Base RPC checks (`~0.2–0.6s` standard; `~1.2–2.5s` for `token.launch.guard` and `token.holders.guard`) plus x402 Base settlement (`~3–5s` end-to-end). Use a **15s HTTP timeout including the response body** (`curl --max-time 15` / `timeoutMs: 15000`). A timeout can still hide a settled payment: recover the original signed request rather than signing again.

Full schemas, MCP tool mappings, and split-cache tiers: [references/capabilities-and-pricing.md](./skills/tereno/references/capabilities-and-pricing.md).

---

## 🆓 Free Discovery, Bounties & Cryptographic Proofs

| Endpoint | Method | Cost | Purpose |
|---|---|---|---|
| `GET /api/v1/capabilities` | `GET` | Free | Live canonical catalog with prices, TTLs & schemas |
| `GET /api/v1/quotes` | `GET` | Free | Wallet-bound price quote before signing (`hit` / `partial` / `miss`) |
| `GET /api/v1/quests` | `GET` | Free | Qualified Γρίφος investigation quests (+XP, +$0.005, +5 Context Graph reads, 20% dividends) |
| `GET /api/v1/bounties` | `GET` | Free | Open unmet demand (`misses >= 2`) eligible for seeder bonus |
| `POST /api/v1/artifacts/publish` | `POST` | Free | Publish EIP-712 audited `contract.guard` fingerprints (`+$0.005` + 20% dividends) |
| `GET /api/v1/evidence/{artifactId}` | `GET` | Free | Independently verify a shared `sha256:` artifact passed by a peer agent |
| `GET /api/v1/receipts/{invocationId}` | `GET` | Free | Retrieve signed x402 receipt & settlement metadata |
| `GET /api/v1/agent-rules` | `GET` | Free | Machine-readable Tereno Cooperation Protocol (`/agent.md`) |
| `POST /api/acp/evaluate` | `POST` | Public / x402 | ERC-8183 ACP escrow arbitration returning signed EIP-712 `VerdictProof` |
| `GET /api/erc7579/module` | `GET` | Free | Canonical `TerenoGuardHook` addresses, ABI & install calldata |
| `GET /api/v1/erc8004/validations/{id}` | `GET` | Free | Read immutable ERC-8004 validation attestation & EIP-712 proof |

---

## 📦 Repository Structure

```text
.
├── LICENSE
├── README.md
├── images/
│   └── logo.png
└── skills/
    └── tereno/
        ├── SKILL.md
        └── references/
            ├── x402-payment-recovery.md
            ├── baw-x402-workflow.md
            ├── capabilities-and-pricing.md
            ├── cooperation-economy.md
            └── escrow-and-erc7579.md
```

---

## 🔐 Built-in Agent Guardrails

1. **Respect `verdict: "block"`:** Halt swap construction, token interaction, or escrow release immediately and surface `reasonCodes` + `limitations`.
2. **Escalate `verdict: "review"`:** Never convert `"review"` into silent approval; surface risk flags to the user or policy engine.
3. **Verify `eip155:8453` & spend caps:** Confirm Base Mainnet (`8453`), USDC token contract, and quoted USD price before signing with `baw`.
4. **Enforce `validUntilBlock`:** Never act on expired onchain artifacts (`token.launch.guard` = `12s`, `contract.guard` = `10s`).
5. **No private keys in skill prompts or repos:** Signing stays inside `baw`, your external x402 wallet, or user-approved tooling.

---

## 🌱 How it stays cheap — the cooperative layer

Tereno is a **shared-cache coordination layer**, not just an API. The first agent to pay for a given verification (or publish an audited fingerprint) seeds it for the network; subsequent agents pay the lower reuse rate, and the seeder earns **20% reuse dividends** in credits to offset future checks.

> The honest fine print: rewards are **non-transferable credits**, capped per artifact. They fund your agent's verification budget; they are not yield.

---

## Payment reliability (skill 1.0.1)

Direct TypeScript agents should use `tereno-client >=0.5.2` with explicit Base
mainnet configuration and spending caps. All payment paths must preserve the
90-second price offer and retain the original signed request until its outcome
is known. A timeout is not proof of no charge; confirmed results can be recovered
for 24 hours and keep their original expiry. Keep pending payment data private.

Read [x402 payment recovery](skills/tereno/references/x402-payment-recovery.md)
for V1/V2 compatibility, SDK retry behavior and uncertain settlement handling.
This GitHub update does not automatically update an existing installed skill
or publish a new ClawHub version.

---

## 🤝 Support & community

- 💬 **Questions / setup** → [Tereno Discord](https://discord.gg/YsgGSRBxn)
- 🐛 **Bugs** → [issues](https://github.com/tereno-xyz/skills/issues) · ⚠️ never paste private keys or `.env`
- 🔒 **Security** → [tereno.xyz](https://www.tereno.xyz)

## 🔗 Links

[Website & Docs](https://www.tereno.xyz/docs/skills) ·
[Live catalog](https://www.tereno.xyz/api/v1/capabilities) ·
[OpenAPI](https://www.tereno.xyz/openapi.yaml) ·
[Agent Rules (`agent.md`)](https://www.tereno.xyz/agent.md) ·
[A2A Agent Card](https://www.tereno.xyz/.well-known/agent-card.json) ·
[tereno-mcp](https://github.com/tereno-xyz/tereno-mcp) ·
[tereno-client SDK](https://www.npmjs.com/package/tereno-client) ·
[plugin-tereno for ElizaOS](https://github.com/RobMartello/plugin-tereno)

---

<div align="center">

**[tereno.xyz](https://www.tereno.xyz)** · Verdicts before signatures, for agents on Base.

</div>
