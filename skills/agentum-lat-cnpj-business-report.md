---
name: Pull a consolidated business-intelligence report on a CNPJ
description: Check a CNPJ for $0.02, then buy the $0.05 consolidated report with activities, signals and an AI summary grounded only in official data.
api: openapi/agentum-lat-apis-brasil-openapi.json
operations: [verificarCnpj, businessIntelligence]
generated: '2026-09-19'
method: generated
inherits: agentum-lat-x402-payment-rules.md
---

# CNPJ business report

## Steps

1. **Digits only.** agentum.lat's contract wants the CNPJ as 14 digits with no punctuation (`"CNPJ brasileiro, 14 dígitos, apenas números"`). Strip the mask before calling.
2. **Confirm the company — `verificarCnpj`** (`GET /verificar-cnpj?cnpj=`, $0.02). Returns `cnpj`, `razao_social`, `situacao`, `data_situacao`, `abertura`, `natureza_juridica`, `uf`, `municipio`, `atividade_principal`, `fonte`. Dates are `dd/mm/yyyy` strings. If `situacao` is not `ATIVA`, decide whether the $0.05 report is still worth buying.
3. **Buy the report — `businessIntelligence`** (`POST /business-intelligence`, JSON body `{"cnpj": "<14 digits>"}`, `Content-Type: application/json`, $0.05). The 200 body has `company`, `registration`, `address`, `activities[]`, `signals[]`, `summary` (a text summary the contract says is generated only from official data, never invented), `sources[]`, `confidence` (a number) and `queriedAt`.
4. **Use the summary as a summary.** Quote the structured fields (`registration`, `activities`, `signals`) as facts and present `summary` as the provider's generated narrative with its `confidence` value attached. The Bazaar example for this route also shows a `partners[]` list (sócios with CPFs masked at source); it is not in the contract's response schema, so handle it as optional.
5. **Cache.** Registry data changes rarely; key the result by CNPJ and `queriedAt` and do not re-buy inside the same session.

## MCP equivalents
`verificar_cnpj` and `business_intelligence` in `@agentum/mcp-server`.
