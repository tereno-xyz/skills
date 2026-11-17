# x402 versions, offered price and payment recovery

Applies to GET/POST invoke routes for Tereno's six canonical capabilities.
Do not assume these recovery guarantees apply to ACP, other paid routes or
another seller. MCP subscriptions are outside this workflow.

## Choose the transport and budget explicitly

For new direct agents use `tereno-client >=0.5.2` or the official V2 x402 clients.
Set `network: "eip155:8453"` for production Base; the Tereno SDK's default is
Base Sepolia. Configure your own maximum budget: the SDK default is $0.05,
while a cold `token.launch.guard` costs $0.07. Raising that cap requires the
user's or agent policy's existing spending authorization.

V2 reads `PAYMENT-REQUIRED` on a 402, sends `PAYMENT-SIGNATURE`, and reads
`PAYMENT-RESPONSE` after settlement. The same challenge also has a V1 JSON body
for legacy clients; its `x402Version: 1` does not replace the V2 header.
The legacy `x402-fetch@0.6.x` / `x402-next@0.6.x` packages in the Vercel starter
are not drop-in replacements for V2 packages. Tereno's scoped compatibility
accepts V1 Base USDC EIP-3009 `exact` envelopes via `X-PAYMENT`, validates them
through the V2 verifier and returns `X-PAYMENT-RESPONSE` on legacy success.
Do not merely rename a V1 payment header to `PAYMENT-SIGNATURE`.

## Hold the server's offered price while signing

Read `X-Tereno-Price-Valid-Until`. An offer is held for 90 seconds, so expiration
of a short-lived cached artifact need not change the amount during signing.
V2 wallets must echo the complete `tereno-offer` extension from the challenge,
preserving its `info` unchanged. The official V2 client used by
`tereno-client >=0.5.2` does this. Check the selected wallet's output rather than
assuming that V2 support alone guarantees extension echoing.

The offer is bound to the exact method, URL/query, canonical JSON body and
claimed wallet header, and may be consumed by one authorization. A V1 client
that cannot echo extensions relies on the same resource, amount and network
origin; changing egress can require a fresh challenge.

If an UNUSED offer expires before any signed request was submitted, obtain a
fresh challenge and re-check its terms. Once a signed request has been sent
and its outcome is uncertain, an expired offer is NOT permission to create a
second authorization. Recover the original payment first.

## Preserve the original signed request

Keep method, URL/query, body bytes, wallet header and signed payment together
in private storage. Do not print signed payment JSON or headers into logs,
chat or telemetry. They are payment credentials. Use private files (mode 0600)
or the runtime's secret storage; do not commit them to Git.

| Observed outcome | Next action |
|---|---|
| 2xx and `PAYMENT-RESPONSE` with `success: true` and transaction | Store receipt and result; check original validity before acting. |
| Timeout, disconnected body, 502/503/504 | Keep budget reserved; retry the exact original signed request. |
| 409 already processing, with `Retry-After` | Wait as directed, then retry the same authorization. |
| `503 payment_recovery_pending` | Keep the original authorization; settlement is uncertain. Never sign another payment to recover it. |
| `409 payment_authorization_scope_mismatch` | Restore the ORIGINAL URL/body/wallet header; changing inputs cannot recover that purchase. |
| New 402 or malformed-payment rejection after an earlier timeout | Reconcile the earlier authorization before treating it as unpaid. |

Bound automatic retries to three attempts, honor `Retry-After`, and then report
pending status for reconciliation. Do not run an indefinite retry/sign loop.
A receipt may already exist even when no payer or transaction could be parsed
from the failed HTTP response.

Confirmed results are recoverable for 24 hours with the original signature.
`X-Tereno-Replayed: true` and `X-Tereno-Original-Response-At` identify recovery;
this does not execute the capability or settle again. The original timestamp,
`validUntilBlock` and expiry remain unchanged. A recovered stale result is not
permission to execute a trade. Only obtain a new paid check after resolving
the old outcome and confirming the need and spending budget for fresh evidence.

When facilitator acknowledgement was lost, the server may need onchain proof.
Unavailable RPC or proof outside its bounded lookup can leave the call pending;
this is not evidence of no charge. Preserve the request for reconciliation.

## SDK recovery

```ts
import { createTerenoClient, TerenoPaymentPendingError } from "tereno-client";

const client = createTerenoClient({
  baseUrl: "https://www.tereno.xyz",
  account, // your existing locally authorized signer; never log its key
  network: "eip155:8453",
  maxPriceUsd: 0.05, // explicit cap; raise to 0.07 only if authorized/needed
  timeoutMs: 15_000,
});

let pendingResume: TerenoPaymentPendingError["resume"] | undefined;
try {
  const result = await client.invokeCapability({ capabilityId: "chain.snapshot" });
  // Store the confirmed result and receipt; validate freshness before acting.
} catch (error) {
  if (!(error instanceof TerenoPaymentPendingError)) throw error;
  pendingResume = error.resume;
  // Retain this closure in the orchestrator, reserve the budget and report pending.
  // End this action here; a later recovery calls pendingResume(), not invokeCapability().
}
```

The SDK already makes up to three HTTP attempts with the same signed payment,
including a timeout for receiving the body. Do not wrap it in a loop that calls
`invokeCapability()` or `buy()` again on each error: that can sign new payments.
Use `error.resume()` for a subsequent recovery attempt and retain any renewed
pending error. This closure is process-local; it does not survive a restart.
For restart-safe recovery, the integration must securely retain the original
signed request. Resolve pending calls before discarding in-memory recovery state.

Public `GET /api/v1/quotes` accepts `wallet` as a query parameter. Do not send
only `x-agent-wallet`: partial agent-auth headers are rejected. Benefits needing
authentication still require the complete signed agent-action headers.

References: [official V1 → V2 migration](https://docs.x402.org/guides/migration-v1-to-v2),
[Vercel starter](https://vercel.com/templates/ai/x402-ai-starter).
