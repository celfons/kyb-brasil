# Example responses

Real answers from the service for one company (ACARÁ AGROHEVEA, CNPJ 04.680.054/0001-04, chosen because it has active IBAMA embargoes, so the risk section is not empty). Captured from a development instance; identical shape in production.

| File | Tool (MCP) | HTTP | Apify input | Price |
|---|---|---|---|---|
| `br_company_check.json` | `br_company_check` | `GET /v1/company?cnpj=` | `tool: company` | free |
| `br_sanctions_screen.json` | `br_sanctions_screen` | `GET /v1/sanctions?cnpj=` | `tool: sanctions` | 0.02 USDC |
| `br_company_dossier.json` | `br_company_dossier` | `GET /v1/dossier?cnpj=` | `tool: dossier` | 0.15 USDC |
| `br_risk_screen.json` | `br_risk_screen` | `GET /v1/risk?cnpj=` | `tool: risk` | 0.05 USDC |
| `br_company_search.json` | `br_company_search` | `GET /v1/search?q=&state=` | `tool: search` | free |
| `br_company_count.json` | `br_company_count` | `GET /v1/count?cnae=&state=&city=…` | (part of `tool: list`) | free |
| `br_company_list.json` | `br_company_list` | `GET /v1/list?cnae=&state=…&page=` | `tool: list` | 0.05 USDC per page of 100 |

How to read the dossier:

- `company` and `partners[]`: Receita Federal registration and ownership (no personal IDs).
- `sanctions`: CGU lists (CEIS, CNEP, CEPIM, leniency) plus the MTE forced-labour list, for the company, its branches and corporate partners. `clear: true` means no active hit.
- `environmental_embargoes`: IBAMA embargo terms aggregated per CNPJ (this company has 23 active ones, hence the 40-point red flag).
- `federal_debt`, `fiscal_regularity`: PGFN active debt and what it implies for a federal clearance certificate (CND).
- `economic_group`: other active companies sharing a partner, each with its own sanction/embargo/debt flags.
- `risk`: deterministic 0-100 score with explained `red_flags[]` in English and Portuguese.
- `data_quality[]`: per section, the reference date of the data and a grade (`fresh`, `aging`, `stale`, `unavailable`, `unknown`).
- `warnings[]`: sections that could not be produced in this run. In these files `FEDERAL_CONTRACTS_NOT_CONFIGURED`, `FEDERAL_DEBT_UNAVAILABLE` and `GAZETTES_UNAVAILABLE` appear because the development instance had no Portal da Transparência key, no PGFN load and no outbound access to Querido Diário; production fills those sections. A section that is `null` is never silently "nothing found".
- `receipt.replay_token`: re-read the same paid answer free within 10 minutes (`br_replay` / `GET /v1/replay?token=`). The tokens in these files are expired.
- `sources[]`: URL, fetch time and reference date of every source used, so each fact can be verified at the origin.

The count and list examples come from a development registry holding a ~1% sample of the Receita Federal dump (18 retail establishments in DF); production answers the same shape over the full registry.

Summaries here are the deterministic template; in production the dossier summary is written by a language model and validated against the JSON (numbers that are not in the data are rejected, falling back to the template).
