---
name: AGENTUM x402 payment and rate-limit rules (shared)
description: The rules every AGENTUM skill inherits — how a call is paid, what a 402 means, what is free, and how to back off.
api: [openapi/agentum-lat-apis-brasil-openapi.json, openapi/agentum-lat-business-openapi.json]
operations: []
generated: '2026-09-19'
method: generated
---

# Shared rules for calling AGENTUM

1. **There is no credential.** No API key, no OAuth, no account. Every paid route answers `402 Payment Required` until the request carries a valid x402 v2 payment (`exact` scheme, USDC on Base mainnet `eip155:8453`). Decode the `PAYMENT-REQUIRED` response header (base64 JSON) to get `accepts[0]` — amount, asset, `payTo`, `maxTimeoutSeconds` (300).
2. **Use a payment-aware client, not hand-rolled signing.** The provider's own path is `npx -y @agentum/mcp-server` with `AGENTUM_MCP_WALLET_KEY` set to the wallet you control; the HTTP equivalent is `@x402/fetch` `wrapFetchWithPayment`. Pin the network to `eip155:8453`, cap the amount per route, and allow only the two published `payTo` wallets (`0xB4f9061e3a6A5533431336506b34e1035029599f` for agentum.lat, `0x7D1EDdfBd167787251fed83b250ABBeA1cf59a6F` for business.agentum.lat).
3. **Prices are fixed and published** (plans/agentum-lat-plans-pricing.yml): $0.01 for most routes, $0.02 for CNPJ/company routes, $0.05 for `businessIntelligence`, $0.15 for `preflight`. The challenge `amount` is in atomic USDC (six decimals): `10000` = $0.01.
4. **Validate locally — the server will not do it for free.** The 402 gate answers before validation: `cnpj=123` and an empty `q=` both returned 402 when probed, so the contract's `400` is only reachable after payment. Check formats yourself: CNPJ 14 digits, CEP 8 digits, CPF 11 digits, LEI 20 characters.
5. **No idempotency, no refunds.** Every call is a paid read; a retry is a second charge and no reversal operation exists (conventions/agentum-lat-conventions.yml). Cache by identifier; responses carry `queriedAt` / `generatedAt` / `as_of`.
6. **Rate limits count unpaid challenges.** `RateLimit-Policy: 10;w=60` on agentum.lat and `120;w=60` on business.agentum.lat; on `429` honour `Retry-After` (rate-limits/agentum-lat-rate-limits.yml). Never poll a 402.
7. **Errors are ad hoc JSON**, not RFC 9457: business host `{"error": CODE, "message": text}`, agentum.lat `{}` on 402 (errors/agentum-lat-problem-types.yml). `502` means an upstream registry did not answer — retry later, it is not a client error.
8. **Language.** Field names and messages are Brazilian Portuguese; agentum.lat uses snake_case, business.agentum.lat camelCase.
