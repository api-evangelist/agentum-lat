---
name: Validate a Brazilian address from a CEP
description: Resolve an 8-digit CEP to street, district, city, state, region and area code for $0.01 before dispatching anything.
api: openapi/agentum-lat-apis-brasil-openapi.json
operations: [verificarCep]
generated: '2026-09-19'
method: generated
inherits: agentum-lat-x402-payment-rules.md
---

# Validate a Brazilian address

## Steps

1. **Normalise the CEP to 8 digits** (`01310-100` becomes `01310100`); the contract wants digits only.
2. **Call `verificarCep`** — `GET /verificar-cep?cep=`, $0.01.
3. **Read the address**: `cep`, `logradouro` (street), `bairro` (district), `municipio`, `uf` (state), `regiao`, `ddd` (telephone area code), `fonte`.
4. **Compare, do not replace.** Match `logradouro`/`municipio`/`uf` against the address you were given and flag mismatches; the response does not carry a street number, so a CEP match confirms the street and city, not the building.
5. **Cache by CEP.** Postal codes are stable; there is no reason to re-buy the same CEP.

## MCP equivalent
`verificar_cep` in `@agentum/mcp-server`.
