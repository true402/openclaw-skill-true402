---
name: true402-token-safety
description: Rug-check a Base token before trading — real buy/sell honeypot simulation plus liquidity and ownership checks. Free daily checks; ~$0.01/call via x402 after.
version: 1.0.0
metadata:
  openclaw:
    emoji: 🛡️
    homepage: https://true402.dev
    requires:
      bins:
        - npx
    envVars:
      PAYER_PRIVATE_KEY:
        required: false
        description: Optional. A Base wallet key holding a little USDC — unlocks unlimited paid checks (~$0.005–0.01 per call over x402, gas sponsored). Leave unset to use the free daily trial.
---

# true402 token safety — rug-check before you trade

Check any Base (chain 8453) ERC-20 token for rug-pull and honeypot risk **before buying, sniping, or approving it**. Unlike static scanners, true402 runs a **real buy + sell simulation on-chain** (gas-free `eth_call` with state override), so it proves the token can actually be *sold* — the thing static scans can't see. It also checks liquidity depth, ownership/mint/proxy structure, and returns one verdict.

## When to use this skill

- The user asks to buy, snipe, ape into, or evaluate a Base token.
- The user asks "is this token safe / legit / a rug / a honeypot?"
- Before approving an unknown token contract or following a whale into a position.
- Inside any autonomous trading loop, as the pre-trade safety gate.

## How to check a token

Run:

```bash
npx -y @true402.dev/rugcheck 0x<token-address>
```

- **No signup, no API key.** The first few checks each day are free (keyless free trial).
- Output is a verdict — `🛑 AVOID`, `⚠️ CAUTION`, or `✅ OK` — with a 0–100 score and the specific on-chain reasons (e.g. "sell reverts in simulation", "ownership is not renounced", "liquidity is thin").
- **Exit code is non-zero when the verdict is AVOID** — safe to use directly as a gate in scripts: `npx -y @true402.dev/rugcheck 0x… && <buy command>`.

## Interpreting the verdict

- `AVOID` — do not buy. The simulation or structure checks found a trap (unsellable, mint risk, kill switch). Tell the user exactly which reasons were flagged.
- `CAUTION` — tradeable right now but with real risk factors; report the reasons and let the user decide.
- `OK` — no traps found on-chain at check time. Not financial advice; liquidity can still be pulled later.

## Unlimited checks (paid, optional)

If `PAYER_PRIVATE_KEY` is set (a Base wallet holding a little USDC), the CLI pays per call over the x402 protocol (~$0.005–0.01, gas sponsored — the wallet needs only USDC). Never print or log the key. If the free trial is exhausted and no key is set, the CLI explains exactly how to enable payment; relay that to the user.

## More checks (same rail, plain HTTP)

Every endpoint answers with HTTP 402 + machine-readable payment terms when unpaid; all are keyless:

```bash
# Full token report (simulation + liquidity + structure + deployer, one verdict)
curl -X POST https://true402.dev/api/v1/base/token-report -H 'content-type: application/json' -d '{"token":"0x…"}'
# Who deployed it — wallet age + fresh-throwaway flag
curl -X POST https://true402.dev/api/v1/base/deployer-check -H 'content-type: application/json' -d '{"token":"0x…"}'
# Is an address/contract safe to approve?
curl -X POST https://true402.dev/api/v1/base/address-safety -H 'content-type: application/json' -d '{"address":"0x…"}'
```

Full catalog: https://true402.dev/api/v1/services · OpenAPI: https://true402.dev/openapi.json · Docs: https://true402.dev/docs/api
