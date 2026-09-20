---
name: Verify a Brazilian counterparty before doing business
description: Escalate from a $0.02 registration check to a $0.15 deterministic preflight verdict on a CNPJ, LEI or company name using AGENTUM Business.
api: openapi/agentum-lat-business-openapi.json
operations: [company, companyIntelligence, preflight]
generated: '2026-09-19'
method: generated
inherits: agentum-lat-x402-payment-rules.md
---

# Verify a Brazilian counterparty

Use this when an agent is about to transact with a Brazilian company (or a foreign one identified by LEI) and needs to know whether it exists, is active, and shows up in public-integrity registries.

## Steps

1. **Normalise the identifier.** A CNPJ is 14 digits (business.agentum.lat accepts it with or without the mask), a LEI is 20 characters, otherwise treat the input as a company name. If you only have a name or LEI, skip to step 4 — only `preflight` accepts those.
2. **Cheapest check first — `company`** (`GET /company?cnpj=`, $0.02). Read `situacaoCadastral` (expect `ATIVA`), `razaoSocial`, `naturezaJuridica`, `cnaePrincipal`, `porte`, `simplesNacional`, `endereco`, and `fonte` (`brasilapi` or `receitaws`). A `400` means the CNPJ was malformed — but it sits behind the payment gate, so check the format before paying. Stop here if all you need is existence and status.
3. **Integrity check — `companyIntelligence`** (`GET /company-intelligence?cnpj=`, $0.02). Returns the same registration plus `findings[]`, each `{source, sourceType (official_api|official_dataset), status, fact, observedAt}` from TCU, CEIS/CNEP, CNJ/CNIA and CVM, with `coverage` saying which sources were actually checked. The contract promises facts only — there is no aggregate score, so do not invent one from the count of findings. A `502` means a registry was down: note which source is missing from `coverage` rather than treating the company as clean.
4. **Consolidated verdict — `preflight`** (`GET /preflight?q=`, $0.15). Accepts CNPJ, LEI or name and returns `band` (`clear` | `flagged` | `insufficient_data`), `confidence` (`high` | `medium` | `low`), `flags[] {code, source, fact}`, `components[]` with per-source `status` (`verified`, `not_found_in_checked_sources`, `unavailable`, `not_applicable`, `conflicting`), `sources[]`, `coverage`, `as_of` and a `disclaimer`. A `404` means no company matched the LEI or name.
5. **Decide.** `clear` with `high` confidence is the only combination that means "checked and nothing found". `insufficient_data` is UNKNOWN, not clean — surface the `components[]` whose status is `unavailable` and retry later. Any `flagged` result should be shown to a human with its `flags[].fact` and `source` verbatim.

## Cost of the full path
$0.02 + $0.02 + $0.15 = $0.19 in USDC per counterparty if all three are run; most flows need only step 2 or only step 4.

## The same capability over other protocols
- MCP: tools `company_intelligence_br` and `preflight` in `@agentum/mcp-server` (no tool for `/company`).
- A2A: skill `company_intelligence` on the AGENTUM Business agent (`https://business.agentum.lat/`, JSON-RPC, send an `A2A-Version: 1.0` header). See a2a/agentum-lat-a2a.yml.
