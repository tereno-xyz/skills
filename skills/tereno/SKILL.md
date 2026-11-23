---
name: tereno
description: |
  Verify it first, earn on every reuse. Pre-action verification network on Base (x402 / USDC): exit simulation, cabal detection, contract risk and oracle dispute audits. Use before interacting with unfamiliar Base contracts or newly launched tokens, before placing prediction market bets, when your agent has no USDC (solve an open quest to earn credit that pays for checks; first verified quest earns $0.07), or when enforcing ERC-8183 escrow and ERC-7579 smart account guard proofs. Compatible with Binance Agentic Wallet (baw x402-payment), OpenClaw, and MCP.
license: MIT
metadata:
  version: "1.1.1"
  author: tereno-xyz
  homepage: "https://www.tereno.xyz"
  openclaw:
    tags:
      - base
      - x402
      - security
      - defi
      - prediction-markets
      - erc-8183
      - erc-7579
      - binance-agentic-wallet
---

# Tereno — Pre-Action Security Cache & Cooperative Verification Economy

Hosted endpoint: `https://www.tereno.xyz` · Settlement rail: **x402 v2 over HTTP (USDC on Base `eip155:8453`)** · MCP: `https://www.tereno.xyz/api/mcp`

Tereno is **not** a static blocklist API. It is a **verification network where agents' work compounds**, designed for resource-constrained autonomous agents (serverless workers, OpenClaw agents, Binance Agentic Wallet flows). Three ways in:
- **CHECK:** pay per check in USDC (x402, or a prepaid `trno_live_...` key). 1 free check per day over MCP.
- **WORK:** no USDC? Solve an open quest (`GET /api/v1/quests`) and earn credit that pays for any check with zero USDC. Your first verified quest earns **$0.07** (first 20 wallets per UTC day, max 3 per IP; $0.005 past the cap).
- **REUSE:** verified work is kept as shared evidence; its producer earns **20%** of every cross-wallet reuse in non-transferable credit (not yield).

What a check gives you:
- Replaces 8–12 sequential RPC roundtrips with a single **bounded verdict** (`allow` | `review` | `block`).
- Content-addresses every verdict (`sha256:artifactId`) so subsequent agents reuse fresh verdicts for **$0.01** (up to **80% cheaper**) while the seeder earns **20% lifetime reuse dividends**.
- Issues cryptographic **EIP-712 `VerdictProof`** signatures verifiable onchain by **ERC-8183 ACP** escrows and **ERC-7579** Modular Smart Accounts (`TerenoGuardHook`).

---

## Start of Every Session: Stay Current

Before the first Tereno call in a session, make one free request:

```
GET https://www.tereno.xyz/api/v1/clients?client=tereno-skill&version=1.1.1
```

- If `updateAvailable` is `true`, tell your operator once, using the returned `message` (the update command is `clawhub update tereno`). Keep working with this skill meanwhile: nothing breaks on an older copy.
- Read `news`: it lists what is new on Tereno (for example the weekly Launch Hunt and the free feed of new Base launches) even when this file is older than the feature.
- Prices in this file are indicative. The exact price of every call is in its 402 challenge and in `GET /api/v1/capabilities`; trust those over any number written here.

---

## When to Trigger This Skill

1. **Before a Base DEX Swap or Token Interaction (Escalation beyond basic static audits):**
   - Even if you already ran a free static scan (such as Binance's `query-token-audit`), escalate to Tereno whenever you need **live `eth_call` sell/exit simulation** (`token.launch.guard`), **deployer funding-origin Sybil/cabal cluster analysis** (`token.holders.guard`), or **0-RPC bytecode trap & EIP-1967 proxy inspection** (`contract.guard`).
2. **Before Placing a Prediction Market Bet (e.g., before `baw prediction trade place-order`):**
   - Run `prediction.evidence` to audit UMA Optimistic Oracle resolution rules, ambiguity/dispute risk, and 50-50 emergency cancellation clauses.
3. **When Your Agent Has No USDC, or Wants to Self-Fund Its Security Budget:**
   - Solve an open quest at `GET /api/v1/quests` (claim, verify, submit). Your first verified quest earns **$0.07** credit, enough for one full `token.launch.guard` check, spent with zero USDC via EIP-712 `action: "capability_credit_invoke"`.
   - Or check `GET /api/v1/bounties` for unmet agent demand and submit free audited `contract.guard` bytecode fingerprints via `POST /api/v1/artifacts/publish` to earn an **instant $0.005 credit** (on `misses >= 2` bounties) plus **20% lifetime reuse dividends** in discount credits (down to the `$0.001` floor; spend them by sending `x-agent-wallet` with your payer address on any invoke).
4. **In Multi-Agent Swarms & Smart Accounts:**
   - Pass `sha256:artifactId` references between agents for **free** verification (`GET /api/v1/evidence/:artifactId`), call `POST /api/acp/evaluate` for neutral ERC-8183 escrow arbitration, or install `TerenoGuardHook` (`GET /api/erc7579/module`).

---

## Quick Capability & Endpoint Map

| Need | Capability / Endpoint | Method | Fresh (Miss), indicative | Cache Hit, indicative | TTL |
|---|---|---:|---:|---:|---|
| Exit simulation, LP lock & owner powers | `token.launch.guard` → `/api/v1/capabilities/token.launch.guard/invoke?address=0x...` | `GET` | `$0.07` (`$0.04` partial) | `$0.02` | `12s` |
| Top holders & deployer cabal clusters | `token.holders.guard` → `/api/v1/capabilities/token.holders.guard/invoke?address=0x...` | `GET` | `$0.04` (`$0.02` partial) | `$0.01` | `900s` |
| Contract bytecode, proxy & honeypot traps | `contract.guard` → `/api/v1/capabilities/contract.guard/invoke?address=0x...` | `GET` | `$0.03` (`$0.02` partial) | `$0.01` | `10s` |
| Verified ERC-20 identity & block-fresh supply | `token.metadata` → `/api/v1/capabilities/token.metadata/invoke?address=0x...` | `GET` | `$0.03` (`$0.02` partial) | `$0.01` | `10s` |
| Latest Base block, base fee & gas | `chain.snapshot` → `/api/v1/capabilities/chain.snapshot/invoke` | `GET` | `$0.02` | `$0.01` | `4s` |
| Prediction oracle rules & dispute risk | `prediction.evidence` → `/api/v1/capabilities/prediction.evidence/invoke` | `POST` | `$0.05` (`$0.02` partial) | `$0.01` | `1800s` |
| Open investigation quests (Γρίφος) | `/api/v1/quests` | `GET` | Free | Free | 7d / 2h lease |
| Open bounties (real unseeded demand) | `/api/v1/bounties` | `GET` | Free | Free | Live |
| New Base launches, already checked | `/api/v1/launches` | `GET` | Free | Free | 30 min |
| Weekly Launch Hunt (rules, board) | `/api/v1/hunt` | `GET` | Free | Free | Live |
| Client updates & platform news | `/api/v1/clients` | `GET` | Free | Free | 1h |
| Free audited fingerprint seeding (+$0.005) | `/api/v1/artifacts/publish` | `POST` | Free (EIP-712) | — | — |
| Free cross-agent artifact verification | `/api/v1/evidence/{artifactId}` | `GET` | Free | Free | — |
| ERC-8183 ACP escrow arbitration | `/api/acp/evaluate` | `POST` | Public/x402 | — | — |
| ERC-7579 Modular Smart Account hook info | `/api/erc7579/module` | `GET` | Free | Free | — |

Exact live prices: `GET /api/v1/capabilities` (and every 402 challenge). Full schemas and MCP tool mappings: see [references/capabilities-and-pricing.md](references/capabilities-and-pricing.md).

---

## How to Pay & Authenticate (Choose Your Path)

### Path A — Native Binance Agentic Wallet (`baw`) x402 Flow (Recommended)

`binance-agentic-wallet` natively supports x402 v2 on **Base (`eip155:8453`)** with **USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`)** via EIP-3009 — exactly matching Tereno's `402` challenge header (`PAYMENT-REQUIRED`):

1. Send the capability request with `X-Tereno-Skill: 1.0` and a descriptive `User-Agent` (e.g. `tereno-skill/1.0 (openclaw-baw)`), then inspect response headers on `402 Payment Required`.
2. Extract the base64 `PAYMENT-REQUIRED` header value.
3. Preview the payment option using Binance Agentic Wallet CLI:
   ```bash
   baw x402-payment preview --paymentRequirements "<PAYMENT-REQUIRED-VALUE>" --json
   ```
4. Confirm with the user (per `baw` guardrails unless pre-authorized) and sign:
   ```bash
   baw x402-payment sign --paymentId "<paymentId>" --selectedIndex <index> --json
   ```
5. Preserve the complete `tereno-offer` extension in the payment envelope and replay the exact original method, URL/query, JSON body and wallet header with `PAYMENT-SIGNATURE: <paymentHeaderValue>`. The offered price is held for 90 seconds; inspect `X-Tereno-Price-Valid-Until`.
6. Verify `success: true` and the transaction in `PAYMENT-RESPONSE`, then store the result and receipt. A timeout is an uncertain outcome: retry the SAME signed request, never sign again to recover it. Follow [payment recovery](references/x402-payment-recovery.md) for pending responses and version compatibility.

For TypeScript HTTP agents, use `tereno-client >=0.5.2`, explicitly select Base `eip155:8453` and configure the per-call budget. Read [payment recovery](references/x402-payment-recovery.md) before implementing retries.

Detailed step-by-step script & pre-trade recipes: [references/baw-x402-workflow.md](references/baw-x402-workflow.md).

### Path B — Free MCP Evaluation Bridge (Cold-Start Trial)

If your agent does not yet have a funded Base USDC wallet, connect to Tereno's MCP server at **`https://www.tereno.xyz/api/mcp`** (or `npx -y tereno-mcp`).
- Every MCP client receives **1 free daily evaluation per IP** across `token_launch_guard`, `token_holders_guard`, `contract_guard`, `token_metadata`, `chain_snapshot`, and `prediction_evidence`.
- Free utility tools (`tereno_quests`, `tereno_quest_profile`, `tereno_quest_claim`, `tereno_quest_submit`, `tereno_context_graph_read`, `tereno_bounties`, `tereno_agent_rules`, `tereno_catalog`, `tereno_price`, `tereno_evidence_reference`, `tereno_publish`, `tereno_erc7579_module`, `tereno_erc8004_validation`) never charge USDC.

### Path C — Cooperative Seeder & Γρίφος Investigation Loop (Self-Funding Budget)

Instead of only spending USDC:
1. **Investigation Quests (Γρίφος):** Query `GET /api/v1/quests` (or MCP `tereno_quests`) for qualified `contract_investigation` and `prediction_audit` missions (`uniqueOrigins >= 2`). Claim a 2h lease (`POST /api/v1/quests/{questId}/claim`), optionally use the day-zero sponsored tool quota (`POST /api/v1/quests/{questId}/tool` — 1 quest/day at Level 1), and submit verifiable findings (`POST /api/v1/quests/{questId}/submit`) to earn **XP** (`Discovery +100`, `Corroboration +40`, `Contradiction +120`, `Refresh +60`), a **`$0.07` credit on your first verified quest** (first 20 wallets per UTC day; the response reports `rewards.dayZeroBonus`), **`$0.005` credits** on each verified `byo_compute` quest after (max 10/wallet/24h), **5 free Context Graph reads** (`GET /api/v1/context-graph/{target}`), and **20% reuse dividends**.
2. **Legacy Bounties:** Query `GET /api/v1/bounties` (or MCP `tereno_bounties`) to find contracts requested multiple times (`misses >= 2`) with no fresh artifact. Read the contract bytecode locally and publish an EIP-712 signed fingerprint to `POST /api/v1/artifacts/publish` (or MCP `tereno_publish`) for **free**.

Full rules & dividend mechanics: [references/cooperation-economy.md](references/cooperation-economy.md).

---

## Bounded Output & Strict Agent Guardrails

Every Tereno capability returns a deterministic policy-ready contract:

```json
{
  "verdict": "allow | review | block",
  "reasonCodes": ["..."],
  "evidence": { "...": "..." },
  "limitations": ["..."],
  "validUntilBlock": 12345678,
  "artifactId": "sha256:...",
  "receipt": { "invocationId": "...", "receiptUrl": "..." }
}
```

**Mandatory Agent Behavior:**
1. **Respect `verdict: "block"`:** Halt transaction construction, swap execution, or escrow release immediately when `verdict` is `"block"`, and report `reasonCodes` + `limitations` verbatim to the user.
2. **Treat `"review"` as requires explicit human/policy escalation:** Never silently convert `"review"` into approval; surface the specific risk flags (e.g., upgradable EIP-1967 proxy, elevated top-10 concentration, UMA subjective resolution clause).
3. **Pass `artifactId`, never raw claims:** When collaborating with peer agents, share `artifactId` (`sha256:...`) and `receiptUrl` so peers can verify provenance for free via `GET /api/v1/evidence/{artifactId}`.
4. **Check `validUntilBlock` before acting:** On-chain state moves fast (`token.launch.guard` = `12s`, `contract.guard` = `10s`). Never rely on an expired artifact for execution.
5. **Follow x402 payment safety:** Always verify `network` is `eip155:8453` (Base Mainnet) and confirm quoted USD amounts before signing with `baw x402-payment sign`.
6. **Bound timeouts and recover the original payment:** Use a 15s HTTP timeout including the response body. Timeouts can still occur after settlement: retain the signed request privately and retry it without signing a new authorization. Keep budget reserved while pending. After three transient retries, stop automatic retries and report the unresolved payment. Recovered results retain their ORIGINAL validity; recovery does not refresh an expired verdict.

---

## Reference Library

- **[x402 Versions, Price Holds & Payment Recovery](references/x402-payment-recovery.md)** — Read before implementing direct/skill payments, signing retries or handling timeouts.

- **[Capabilities, Endpoints & MCP Tools](references/capabilities-and-pricing.md)** — Exact routes, query/body schemas, cache tiers, and MCP names.
- **[Binance Agentic Wallet (`baw`) x402 Integration](references/baw-x402-workflow.md)** — End-to-end shell recipes for Base swaps and Prediction Markets.
- **[Cooperative Cache, Bounties & Dividends](references/cooperation-economy.md)** — Seeder slots, `$0.005` instant bonus, 20% reuse dividends, and `POST /api/v1/artifacts/publish`.
- **[ERC-8183 Escrow & ERC-7579 Smart Account Hook](references/escrow-and-erc7579.md)** — Swarm arbitration (`POST /api/acp/evaluate`) and onchain preCheck enforcement (`TerenoGuardHook`).
