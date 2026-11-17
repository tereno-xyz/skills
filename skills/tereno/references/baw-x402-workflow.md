# Binance Agentic Wallet (`baw`) + Tereno x402 Workflow

`binance-agentic-wallet` (`baw`) natively supports **x402 v2** payments on **Base (`eip155:8453`)** with **USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`)** via `eip3009`. This allows any Binance Agent OS / OpenClaw agent to pay Tereno directly from its existing agentic wallet without installing custom SDKs or exposing raw private keys.

---

## Recipe 1: End-to-End `baw` x402 Request & Payment Loop

Use this exact sequence for any paid Tereno capability (`token.launch.guard`, `token.holders.guard`, `contract.guard`, `token.metadata`, `chain.snapshot`, or `prediction.evidence`):

### Step 1 — Trigger the 402 Challenge & Keep Private Request State

Requires `curl`, `jq` and an already configured `baw` wallet. Set `TARGET_URL`
from the actual task. This GET example uses `token.launch.guard`; a POST must
use the same method and body file on both the challenge and every paid replay.

```bash
umask 077
TERENO_PAYMENT_DIR=$(mktemp -d)
TARGET_URL="https://www.tereno.xyz/api/v1/capabilities/token.launch.guard/invoke?address=0x..."
printf '%s' "$TARGET_URL" > "$TERENO_PAYMENT_DIR/url.txt"

TERENO_STATUS=$(curl -sS --max-time 15 \
  -D "$TERENO_PAYMENT_DIR/challenge-headers.txt" \
  -o "$TERENO_PAYMENT_DIR/challenge-body.json" -w '%{http_code}' \
  -H "User-Agent: tereno-skill/1.0.1 (openclaw-baw)" \
  -H "X-Tereno-Skill: 1.0" "$TARGET_URL")
[ "$TERENO_STATUS" = "402" ] || { printf 'Expected a 402 challenge; stop before signing.\n' >&2; exit 1; }
TERENO_PAYMENT_REQUIRED=$(awk 'tolower($0) ~ /^payment-required:/ { sub(/^[^:]+:[[:space:]]*/, ""); sub(/\r$/, ""); print }' "$TERENO_PAYMENT_DIR/challenge-headers.txt")
[ -n "$TERENO_PAYMENT_REQUIRED" ] || { printf 'Missing V2 payment header; stop before signing.\n' >&2; exit 1; }
```

Inspect `X-Tereno-Price-Valid-Until`. V2 clients must use `PAYMENT-REQUIRED`,
even though the compatibility response also has a V1 JSON body.

### Step 2 — Preview the Offered Terms

```bash
baw x402-payment preview --paymentRequirements "$TERENO_PAYMENT_REQUIRED" --json \
  > "$TERENO_PAYMENT_DIR/preview.json"
```

Inspect only the fields needed to choose the payment:
- `data.options[i].status` is `"READY_TO_SIGN"`.
- `data.options[i].binanceChainId` is `"8453"` (Base Mainnet).
- Asset is Base USDC `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`.
- `data.options[i].amountUsd` is within the caller's authorized per-call AND
  remaining daily budget. The current ordinary range is $0.01–$0.07.
- Destination and `exact` / EIP-3009 terms match the server's challenge.

Extract `PAYMENT_ID` and `SELECTED_INDEX` from the preview. Do not guess the
index. If an UNUSED offer expires before submitting a signed payment, fetch a
fresh challenge and preview it again.

### Step 3 — Sign Once & Retain the Authorization Privately

Use existing spending authorization, or obtain confirmation as required by the
wallet's guardrails. Never sign again merely because a submitted call timed out.

```bash
baw x402-payment sign --paymentId "$PAYMENT_ID" --selectedIndex "$SELECTED_INDEX" --json \
  > "$TERENO_PAYMENT_DIR/signed.json"
TERENO_HEADER_NAME=$(jq -er '.data.paymentHeaderName' "$TERENO_PAYMENT_DIR/signed.json")
TERENO_HEADER_VALUE=$(jq -er '.data.paymentHeaderValue' "$TERENO_PAYMENT_DIR/signed.json")
[ "$(printf '%s' "$TERENO_HEADER_NAME" | tr '[:upper:]' '[:lower:]')" = "payment-signature" ] || { printf 'Expected V2 PAYMENT-SIGNATURE; inspect wallet configuration.\n' >&2; exit 1; }
printf 'User-Agent: tereno-skill/1.0.1 (openclaw-baw)\nX-Tereno-Skill: 1.0\nPAYMENT-SIGNATURE: %s\n' "$TERENO_HEADER_VALUE" \
  > "$TERENO_PAYMENT_DIR/paid-request-headers.txt"
```

Do not echo `signed.json` or the payment header. Before sending, inspect the
V2 envelope privately and verify that it preserves the challenge's complete
`extensions["tereno-offer"]` unchanged. Without this extension the wallet does
not obtain the 90s price hold. If the installed wallet drops extensions, stop
before submission and use a client that preserves them; do not assume native
V2 support proves this behavior. See [payment recovery](x402-payment-recovery.md).

Base USDC EIP-3009 normally needs no approval transaction. If the wallet instead
proposes Permit2 or an approval, re-check that the payment scheme and additional
transaction are supported and authorized before proceeding; this recipe covers
EIP-3009 exact payments only.

### Step 4 — Submit & Recover the SAME Signed Request

```bash
TERENO_STATUS=$(curl -sS --max-time 15 \
  -D "$TERENO_PAYMENT_DIR/response-headers.txt" \
  -o "$TERENO_PAYMENT_DIR/response-body.json" -w '%{http_code}' \
  -H "@$TERENO_PAYMENT_DIR/paid-request-headers.txt" \
  "$(cat "$TERENO_PAYMENT_DIR/url.txt")") || TERENO_STATUS=timeout
printf 'HTTP outcome: %s\n' "$TERENO_STATUS"
```

On timeout, incomplete body, in-progress 409 or `503 payment_recovery_pending`,
keep the authorization and budget reservation. Re-run **Step 4 only**, honoring
`Retry-After`; do not repeat signing. Bound automatic retries to three and then
report pending status for reconciliation. A timeout can occur after settlement.
Keep the request files until the outcome is resolved; do not delete them in an
error cleanup hook. Use durable private storage if recovery must survive a
process restart or temporary-directory cleanup.

For POST `prediction.evidence`, use `--request POST --header 'Content-Type: application/json'`
and `--data-binary @<original-private-body-file>` on both unsigned and signed
requests. Preserve the original URL/query, body and any wallet header throughout.

On confirmed success, privately decode `PAYMENT-RESPONSE` and require
`success: true` plus `transaction`. A recovered response has
`X-Tereno-Replayed: true` and its ORIGINAL validity window. Check the result's
`verdict`, limitations and expiry before acting; recovery does not refresh a
stale security verdict. See [payment recovery](x402-payment-recovery.md) for
version compatibility, pending errors and the 24h confirmed-result recovery window.

---

## Recipe 2: Safe Base DEX Swap Pipeline (Binance + Tereno)

Combine Binance's free discovery/audit skills with Tereno's deep pre-execution guards:

1. **Initial Static Scan (Free):** Run Binance `query-token-audit` (`chainId: 8453`) to filter obvious static red flags.
2. **Exit Simulation & Owner Powers (Tereno):** Run `token.launch.guard` (`GET /api/v1/capabilities/token.launch.guard/invoke?address=<token>`) via `baw` x402 to verify via live `eth_call` that a buy can actually be sold on the DEX pool without hidden tax/blacklist traps.
3. **Insider / Sybil Cluster Check (Tereno):** For newly launched or low-cap tokens, run `token.holders.guard` (`GET /api/v1/capabilities/token.holders.guard/invoke?address=<token>`) to check whether top holders were funded by the same deployer/funder cabal.
4. **Policy Gate:**
   - If either verdict returns `"block"`, **abort the swap** and present `reasonCodes` to the user.
   - If `"review"`, surface `evidence` and `limitations` and ask for explicit confirmation.
   - Only if `"allow"`, proceed with swap quote/execution.

---

## Recipe 3: Safe Prediction Market Pipeline (`prediction.evidence` + `baw`)

Before running `baw prediction trade quote` or `baw prediction trade place-order`:

1. Send `POST https://www.tereno.xyz/api/v1/capabilities/prediction.evidence/invoke` with:
   ```json
   {
     "question": "<Exact market proposition>",
     "market": "polymarket",
     "marketUrl": "<Optional market URL>"
   }
   ```
2. Pay the HTTP `402` challenge using `baw x402-payment preview` + `sign` (if cached by another agent, cost drops to **`$0.01`** vs `$0.05` fresh).
3. Inspect the returned oracle audit:
   - Check UMA Optimistic Oracle dispute risk and ambiguous wording alerts.
   - Check for emergency 50-50 / invalidation resolution clauses.
   - Only proceed to `baw prediction trade place-order` if `verdict` is acceptable and `validUntilBlock` is current.
