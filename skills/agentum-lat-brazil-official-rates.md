---
name: Fetch official Brazilian rates (Selic, CDI, commercial dollar)
description: One $0.01 call returns the Banco Central SGS Selic target, CDI daily rate and commercial-dollar sell rate for BRL arithmetic.
api: openapi/agentum-lat-apis-brasil-openapi.json
operations: [taxasBrasil]
generated: '2026-09-19'
method: generated
inherits: agentum-lat-x402-payment-rules.md
---

# Official Brazilian rates

## Steps

1. **Call `taxasBrasil`** — `GET /taxas-brasil`, no parameters, $0.01.
2. **Read three series**, each an object `{valor, data, unidade}`: `selic_meta_aa` (Selic target, `% ao ano`), `cdi_ad` (CDI, `% ao dia`), `dolar_comercial_venda` (commercial dollar sell rate, `BRL por USD`). `fonte` names the source (`Banco Central do Brasil (SGS)`).
3. **Parse carefully.** `valor` is a STRING (`"14.00"`, `"0.051660"`, `"5.1253"`) and `data` is `dd/mm/yyyy`. Convert before arithmetic and always carry `unidade` — CDI is per day, Selic per year.
4. **Respect the data date, not the call time.** Each series carries its own `data`; the dollar rate can be a day or more older than the Selic decision. Cache for the trading day and re-buy only when the date matters.
5. **Need other currencies?** `GET /fx-rates?base=&symbols=` ($0.01, ECB via Frankfurter, ISO 4217 codes) is documented in llms.txt and served live, but it is NOT in the published OpenAPI — its input shape is in the Bazaar schema of its 402 challenge and in the MCP tool `fx_rates`.

## MCP equivalent
`taxas_brasil` (no input) in `@agentum/mcp-server`.
