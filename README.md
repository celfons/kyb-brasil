<p align="center">
  <img src="assets/icon.png" width="96" height="96" alt="KYB Brasil">
</p>

<h1 align="center">KYB Brasil</h1>

<p align="center">
  <b>Due diligence on Brazilian companies for AI agents, in one call.</b><br>
  Registration, ownership, sanctions, forced-labour list, federal debt, environmental embargoes, federal contracts, BNDES financing, OFAC/UN/EU screening and an explained risk score, from official sources.<br>
  Pay per call with <a href="https://x402.org">x402</a> (USDC). No signup, no API key.
</p>

<p align="center">
  <a href="https://kyb-br-agents.marcelfons.workers.dev/mcp">MCP endpoint</a> ·
  <a href="https://kyb-br-agents.marcelfons.workers.dev/llms.txt">llms.txt</a> ·
  <a href="https://kyb-br-agents.marcelfons.workers.dev/terms">Terms</a> ·
  <a href="https://registry.modelcontextprotocol.io/v0/servers?search=io.github.celfons/kyb-brasil">MCP Registry</a>
</p>

---

## Why

Checking a Brazilian counterparty means stitching together half a dozen government sources with different formats: the Receita Federal company registry, the CGU sanction lists, the federal contracts portal. Agents that onboard suppliers, approve marketplace sellers or vet partners in Brazil either skip the check or make ten calls and get something wrong.

KYB Brasil does the stitching. Send a CNPJ (the Brazilian company ID); get back structured JSON, a summary in English and Portuguese, and the source URL and fetch time of every fact.

## Tools

| Tool | What you get | Price (USDC) |
|---|---|---|
| `br_company_check` | Receita Federal registration: status (ATIVA, BAIXADA, INAPTA…), legal name, main activity (CNAE), size, age, normalized address | **free** (50/day) |
| `br_sanctions_screen` | The company, **all its branches** and its **corporate partners** screened against the official CGU lists: CEIS (debarred), CNEP (anti-corruption penalties), CEPIM (barred non-profits), leniency agreements, plus the **MTE forced-labour employer list**. Each hit has type, issuing body, dates, active flag and an evidence URL | 0.03 |
| `br_company_dossier` | Everything above plus ownership (no personal IDs), federal government contracts, **active federal debt** (PGFN), **environmental embargoes** (IBAMA), BNDES financing, **OFAC/UN/EU** name screening, municipal gazette mentions, and a deterministic **0-100 risk score** with explained red flags | 0.15 |

Every answer includes `summary.en`, `summary.pt`, `sources[]`, `data_freshness`, `generated_at` and `cached`. Errors (invalid or unknown CNPJ, source unavailable) are **never charged**.

## Use it

### MCP (Claude, Cursor, any MCP client)

Remote server, streamable HTTP:

```
https://kyb-br-agents.marcelfons.workers.dev/mcp
```

```bash
claude mcp add --transport http kyb-brasil https://kyb-br-agents.marcelfons.workers.dev/mcp
```

Claude Desktop / Cursor (`mcpServers` config):

```json
{
  "mcpServers": {
    "kyb-brasil": {
      "url": "https://kyb-br-agents.marcelfons.workers.dev/mcp"
    }
  }
}
```

The free tool works in any client. Paid tools answer with an x402 `PAYMENT_REQUIRED` error that x402-aware clients (Cloudflare Agents SDK `withX402Client`, Coinbase AgentKit, MCP payment proxies) pay and retry automatically.

### HTTP

```bash
# free
curl "https://kyb-br-agents.marcelfons.workers.dev/v1/company?cnpj=33.000.167/0001-01"

# paid: answers HTTP 402 with a PAYMENT-REQUIRED header (base64 JSON: price, network, payTo)
curl -i "https://kyb-br-agents.marcelfons.workers.dev/v1/dossier?cnpj=33.000.167/0001-01"
curl -i "https://kyb-br-agents.marcelfons.workers.dev/v1/sanctions?cnpj=33.000.167/0001-01"
```

Paying from code with the official x402 client:

```ts
import { x402Client } from "@x402/core/client";
import { registerExactEvmScheme } from "@x402/evm/exact/client";
import { toClientEvmSigner } from "@x402/evm";
import { wrapFetchWithPayment } from "@x402/fetch";
import { privateKeyToAccount } from "viem/accounts";

const client = new x402Client();
registerExactEvmScheme(client, { signer: toClientEvmSigner(privateKeyToAccount(process.env.PK as `0x${string}`)) });
const fetchWithPay = wrapFetchWithPayment(fetch, client);

const res = await fetchWithPay("https://kyb-br-agents.marcelfons.workers.dev/v1/dossier?cnpj=33.000.167/0001-01");
console.log(await res.json());
```

Payments settle in USDC on Base through the Coinbase facilitator. The payer needs USDC only (no ETH for gas).

### Apify

Prefer a card and a spreadsheet? The same reports run in batch (up to 500 CNPJs per run) as an Actor on the Apify Store, billed per result.

## Example (real response, trimmed)

`GET /v1/company?cnpj=33.000.167/0001-01`

```json
{
  "cnpj": "33000167000101",
  "cnpj_formatted": "33.000.167/0001-01",
  "legal_name": "PETROLEO BRASILEIRO S A PETROBRAS",
  "registration_status": "ATIVA",
  "opened_at": "1966-09-28",
  "age_years": 60,
  "main_activity": { "code": "0600-0/01", "description": "Extração de petróleo e gás natural" },
  "size": "DEMAIS",
  "is_headquarters": true,
  "address": { "city": "RIO DE JANEIRO", "state": "RJ", "country": "BR" },
  "summary": {
    "en": "PETROLEO BRASILEIRO S A PETROBRAS (CNPJ 33.000.167/0001-01) is ATIVA at the Receita Federal since 2005-11-03. Headquarters opened 1966-09-28 (60 years), main activity 0600-0/01 Extração de petróleo e gás natural, size DEMAIS, based in RIO DE JANEIRO/RJ.",
    "pt": "PETROLEO BRASILEIRO S A PETROBRAS (CNPJ 33.000.167/0001-01) está ATIVA na Receita Federal desde 2005-11-03. Matriz aberta em 1966-09-28 (60 anos), atividade principal 0600-0/01 Extração de petróleo e gás natural, porte DEMAIS, sediada em RIO DE JANEIRO/RJ."
  },
  "sources": [
    { "name": "Receita Federal (Dados Abertos CNPJ) via BrasilAPI", "url": "https://brasilapi.com.br/api/cnpj/v1/33000167000101", "fetched_at": "2026-10-11T19:02:16.208Z" }
  ],
  "data_freshness": "Receita Federal open data is refreshed monthly; status changes may lag up to ~45 days.",
  "generated_at": "2026-10-11T19:45:39.682Z",
  "cached": true
}
```

The dossier adds `partners[]`, `sanctions` (hits with evidence URLs), `federal_contracts` (count, total value, sample), `federal_debt` (PGFN: inscriptions, total BRL, judicial, statuses), `environmental_embargoes` (IBAMA: active, total, area in ha, municipalities), `bndes_financing` (contracts, total BRL), `international_sanctions` (exact name matches on OFAC/UN/EU, informational), `gazette_mentions` (latest municipal gazette mentions), `risk` (`level`, `score`, `red_flags[]` with `title`/`detail` in English and Portuguese) and `warnings[]`. A section is `null` with a warning when its source is unavailable, never "nothing on record".

## Risk score

Deterministic: the same data always gives the same score. Points add up to a 0-100 score; level is `low` (<20), `medium` (20-49), `high` (50-79) or `critical` (80+).

| Red flag | Points |
|---|---|
| Active sanction on this CNPJ | 80 |
| Listed on the MTE forced-labour employer list | 80 |
| Registration not active (BAIXADA, INAPTA, SUSPENSA, NULA) | 60 |
| Active sanction on another branch of the same company | 40 |
| Active sanction on a corporate partner | 30 |
| Active IBAMA environmental embargo | 40 |
| Active federal debt at the PGFN (+20 if above R$ 1 million) | 20 |
| Opened less than 12 months ago (+10 if less than 3 months) | 15 |
| Tiny share capital vs. large federal contracts | 15 |
| Past (expired) sanctions only | 10 |
| No partners listed where the legal form requires them | 10 |
| Some source unavailable (section returned null with a warning) | 5 |

Foreign partners, MEI status, BNDES financing, gazette mentions and possible OFAC/UN/EU name matches are reported as information (0 points).

## Data sources

| Source | Used for | Freshness |
|---|---|---|
| Receita Federal open CNPJ data (via BrasilAPI) | registration, ownership | monthly dump |
| CGU / Portal da Transparência: CEIS, CNEP, CEPIM, leniency agreements | sanctions | re-ingested daily |
| Portal da Transparência: federal contracts | contracts | live |
| MTE: Cadastro de Empregadores (forced labour) | sanctions | re-ingested weekly |
| PGFN: Dívida Ativa da União (open data) | federal debt | re-ingested monthly |
| IBAMA: embargo terms | environmental embargoes | re-ingested daily |
| BNDES: direct financing operations | financing | re-ingested monthly |
| OFAC SDN, UN Security Council, EU consolidated list (entities only) | name screening | re-ingested daily |
| Querido Diário: municipal official gazettes | gazette mentions | live |

Only official open data. No scraping behind logins, no CAPTCHA bypassing.

## Privacy (LGPD)

Built for company verification (KYB), not for profiling people. CPF numbers are never stored or returned, sanctions are never matched against individuals, natural-person rows are dropped from every list at ingestion, and international lists are matched against company names only. Cached answers expire in 24 hours. Details and contact: [terms](https://kyb-br-agents.marcelfons.workers.dev/terms).

## Status

Production. Runs on Cloudflare Workers. Not legal or compliance advice: verify critical facts at the source URLs included in every answer.

---

<sub>KYB Brasil · due diligence de empresas brasileiras para agentes de IA · pagamento por chamada via x402</sub>
