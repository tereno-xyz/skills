# Tereno Agent Skills (`tereno-xyz/skills`)

Official Agent Skills for **[Tereno](https://www.tereno.xyz)** — a sub-second pre-action security cache, cooperative verification economy, and cryptographic proof layer for autonomous agents on **Base** (`eip155:8453` / x402 USDC).

Designed for **OpenClaw**, **Binance Agentic Wallet (`baw`)**, **Claude Code**, **Cursor**, **Gemini CLI**, and any agent runtime supporting the [Agent Skills (`SKILL.md`)](https://agentskills.io) standard.

---

## Install

```bash
npx skills add tereno-xyz/skills
```

Or install directly via OpenClaw / ClawHub once published:

```bash
openclaw skills install tereno
```

---

## What the `tereno` Skill Gives Your Agent

Instead of wrapping a single static endpoint, this skill equips your agent with **Tereno's entire cooperative security & verification engine**:

1. **Pre-Execution Token & Launch Safety (`token.launch.guard`, `token.holders.guard`, `contract.guard`)**
   - Live `eth_call` buy/sell exit simulation on Base DEX pools, owner power inspection, and bytecode trap detection.
   - Deployer funding-origin graph & Sybil/cabal cluster analysis before entering a position.
2. **Prediction Market Oracle Risk Audit (`prediction.evidence`)**
   - Audits UMA Optimistic Oracle resolution rules, ambiguity risk, and emergency 50-50 clauses before placing bets (pairs directly with `baw prediction trade`).
3. **Native Binance Agentic Wallet (`baw`) x402 Payment Flow**
   - Step-by-step recipe to pay HTTP `402 Payment Required` challenges on Base USDC (`eip155:8453`) natively using `baw x402-payment preview` and `baw x402-payment sign` — plus a **1 free daily MCP evaluation** bridge for cold starts.
4. **Cooperative Cache & Self-Funding Agent Economics**
   - **Cache-First ($0.01):** Reuse fresh verified verdicts up to 80% cheaper than cold computation.
   - **Open Bounties & +$0.005 Seeder Bonus:** Query `/api/v1/bounties` and publish audited `contract.guard` fingerprints for free via `/api/v1/artifacts/publish`.
   - **20% Lifetime Reuse Dividends:** Earn non-transferable credits whenever peer wallets reuse verdicts you seeded, discounting your future calls down to `$0.001`.
   - **Free Provenance Verification:** Share `sha256:artifactId` links across agents (`GET /api/v1/evidence/:id`).
5. **Cryptographic Swarm Arbitration & Smart Account Guards (`ERC-8183` & `ERC-7579`)**
   - Neutral ACP escrow evaluation (`POST /api/acp/evaluate`) returning EIP-712 `VerdictProof` signatures, plus `TerenoGuardHook` enforcement for Safe / Kernel / Biconomy / Coinbase Smart Accounts (`GET /api/erc7579/module`).

---

## Repository Structure

```text
.
├── LICENSE
├── README.md
└── skills/
    └── tereno/
        ├── SKILL.md
        └── references/
            ├── baw-x402-workflow.md
            ├── capabilities-and-pricing.md
            ├── cooperation-economy.md
            └── escrow-and-erc7579.md
```

---

## Related Packages & Discovery

- **Website & Live Docs:** [https://www.tereno.xyz](https://www.tereno.xyz)
- **MCP Server (`tereno-mcp`):** `npx -y tereno-mcp` or `https://www.tereno.xyz/api/mcp` ([github.com/tereno-xyz/tereno-mcp](https://github.com/tereno-xyz/tereno-mcp))
- **Agent Rules Protocol:** `https://www.tereno.xyz/agent.md` (`GET /api/v1/agent-rules`)
- **A2A Agent Card:** `https://www.tereno.xyz/.well-known/agent-card.json`
- **x402 Manifest:** `https://www.tereno.xyz/.well-known/x402`
- **ERC-8004 Canonical Identity:** `agentId: 63721` on Base Identity Registry
