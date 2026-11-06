# Merit — Ukrainian Law & Registers (MCP server)

A remote [Model Context Protocol](https://modelcontextprotocol.io) server that gives an AI assistant
read-only access to Ukrainian law and public registers: legislation in its version on any date,
Supreme Court practice, court hearings, the company register, professional registers, enforcement
proceedings, debtors and five sanctions lists.

- **Endpoint:** `https://law.merit.legal/mcp` (Streamable HTTP)
- **Registry name:** `legal.merit/law`
- **Authentication:** OAuth 2.1 — the client opens the Merit sign-in page, you sign in with your email
- **Website, pricing and help:** <https://merit.legal/en/mcp>

> This repository holds documentation only. The server is a hosted service; its source code is not public.

## What it does

The assistant calls the server instead of answering from memory. It searches the official sources,
reads the right article or record and answers with the source and its date.

- **Read only.** No tool changes or sends anything.
- **It does not see your chat.** The assistant sends only the request to a source (a case number,
  an article, a company code).
- **Stateless.** Each request stands alone.

### Sources

50+ sources and 120+ tools. Each source is a set of tools named `source__action`
(for example `ua_law__get_unit_in_force`). The assistant sees the main tools at once and finds the
others through the server's catalogue tools (`catalog__search`, `catalog__describe`, `catalog__call`).

| Group | Sources |
|---|---|
| Legislation | `ua_law` (Verkhovna Rada acts, editions on any date, comparison of editions, draft laws), `ua_npa_registry`, `ua_ccu`, `ua_dstu`, `ua_war_territories` |
| Case law | `ua_edrsr` (Unified State Register of Court Decisions), `ua_lpd` (Supreme Court legal positions), `ua_vs_digest`, `ua_echr` |
| Courts | `ua_court_directory`, `ua_court_hearings`, `ua_court_transfer` |
| Companies and public data | `ua_edr_companies` (companies and sole proprietors), `ua_nazk_declarations`, `ua_corruption_registry`, `ua_licenses`, `ua_media_registry`, `prozorro`, `ua_construction`, `ua_ip` |
| Tax | `ua_tax_registers`, `ua_zir` |
| Foreign companies | `gleif`, `uk_companies_house`, `icij_offshore_leaks` |
| Person checks | `ua_wanted_persons`, `ua_lustration`, `ua_lost_documents` |
| Professional registers | `ua_advocates`, `ua_notaries`, `ua_experts`, `ua_expert_methodics`, `ua_trustees`, `ua_auditors`, `ua_appraisers` |
| Debts, enforcement, bankruptcy | `ua_enforcement_proceedings`, `ua_debtors`, `ua_bankruptcy` |
| Sanctions | `ua_sanctions`, `eu_sanctions`, `ofac_sanctions`, `uk_sanctions`, `un_sanctions` |
| Regulators | `ua_amcu_decisions`, `ua_amcu_offenders`, `ua_nbu_bank_directory`, `ua_nbu_financial_institutions`, `ua_nbu_currency_exchange` |
| Calculations and calendar | `ua_nbu_rate`, `ua_cpi_inflation`, `ua_legal_indicators` (court fee, subsistence minimum, NBU rate), `ua_holidays`, `ua_gp_crime_stats` |

### Honest answers

Every source follows the same rules:

- `as_of` is the publication date of the source itself, not the date of our sync. An outdated snapshot carries `stale`.
- `available: false` with a `reason` means the source could not answer. It is not an empty result.
- An empty result never means that the person, company or proceeding does not exist.
- A shortened field always carries a flag or a character count.
- Sanctions are split by authority. A "no" from one list says nothing about the others.

## Configuration

You need a Merit account: <https://merit.legal/en/mcp>. On the first call the client opens the sign-in page.

### VS Code (GitHub Copilot)

Add to `.vscode/mcp.json` or to your user MCP configuration:

```json
{
  "servers": {
    "merit": {
      "type": "http",
      "url": "https://law.merit.legal/mcp"
    }
  }
}
```

### Cursor and other clients with `mcpServers`

```json
{
  "mcpServers": {
    "merit": {
      "url": "https://law.merit.legal/mcp"
    }
  }
}
```

### Claude, ChatGPT, Gemini

- **Claude** (claude.ai, Desktop, mobile): Customize → Connectors → find Merit, or add a custom connector with `https://law.merit.legal/mcp`.
- **ChatGPT:** find Merit under Plugins, or add a custom MCP server with the same address.
- **Gemini:** add a custom app with the same address in the connected apps settings.

## Usage

Ask a legal question in plain language, in Ukrainian or English. The assistant picks the tools.

| Ask | Tools the assistant uses |
|---|---|
| Which version of Art. 625 of the Civil Code was in force on 15 March 2022, and what has changed since? | `ua_law__get_unit_in_force`, `ua_law__compare_unit_editions` |
| Find Supreme Court positions on 3% per annum and inflation losses under martial law. | `ua_lpd__search`, `ua_vs_digest__search`, `ua_edrsr__search` |
| Check a counterparty by its EDRPOU code: status, director, enforcement proceedings, sanctions. | `ua_edr_companies__lookup`, `ua_enforcement_proceedings__search`, `ua_debtors__search`, `ua_sanctions__search`, `eu_sanctions__search` |
| Calculate inflation losses on a debt of 120,000 UAH from February 2024 to September 2026. | `ua_cpi_inflation__lookup`, `ua_law__get_unit_in_force` |
| When is the next hearing in my case, and has its jurisdiction changed? | `ua_court_hearings__search`, `ua_court_transfer__search`, `ua_court_directory__lookup` |
| Is the certificate of the other side's advocate valid? | `ua_advocates__search`, `ua_advocates__get_advocate` |

The server gives the assistant sources with links and dates. The conclusion and the responsibility stay with the lawyer.

## Plans

A trial week costs 1 UAH. Paid plans start at 790 UAH per 4 weeks. A call is one successful tool call.
Details: <https://merit.legal/en/mcp#pricing>.

## Support

- Email: hello@merit.legal
- Help: <https://merit.legal/en/mcp/help>
- Terms (Ukrainian): <https://merit.legal/mcp/offer>
