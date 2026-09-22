# scraper-js — JS scraper template

The Node.js half of the [Brewtality-3-16](../README.md) self-healing job-scraper
template for [peviitor.ro](https://peviitor.ro). `node-fetch` + Cheerio, ESM,
Jest.

> **This is a template.** `scraper/config/*.json`, `docs/`, `ai/` and the
> workflows ship `{{PLACEHOLDER}}` values. To make a real scraper, copy this
> folder into a new repo and replace them — see the [placeholder list](#placeholders).

## What it does (once configured)

1. **Validate the company** via the public ANAF API (`demoanaf.ro`) by CIF — name, active/inactive status, address. Cached in `tmp/company.json` (7-day TTL) with a stale-cache fallback.
2. **Scrape jobs** from the company's own careers listing (HTTP + Cheerio, no browser), reconciled against its job sitemap, plus ANOFM by CIF.
3. **Self-heal** every field through a selector cascade (primary CSS → fallback CSS → structural / JSON-LD → regex). See [`ai/AGENTS.md`](ai/AGENTS.md).
4. **Validate + canary** — drop jobs with a bad URL / empty title; abort before any write if the scrape produced nothing.
5. **Upsert** to the Peviitor API (retry + backoff on transient failures).
6. **Generate** `docs/jobs.md` and refresh `docs/company.json` for GitHub Pages.

## Quick start

```bash
npm install
npm run test:unit        # 131 tests — pass with placeholders in place
npm run scrape           # runs the full pipeline (no-op until configured)
```

## Placeholders

| Placeholder | Fill with |
|---|---|
| `TURBOMECANICA SA` | legal name, uppercase (e.g. `EXAMPLE COMPANY SRL`) |
| `Turbomecanica` | commercial brand |
| `3156315` | fiscal code (CUI), no `RO` prefix |
| `https://turbomecanica.ro` | `https://www.example.com` |
| `https://turbomecanica.ro/en/careers/available-jobs/` | the open-positions listing page |
| `` | the job sitemap URL (or `""` if the site has none) |
| `https://turbomecanica.ro/job/` | canonical job-permalink prefix, e.g. `https://www.example.com/jobs/` |
| `Bucuresti` | HQ city (falls back to `România` in the transform) |
| `article.job-preview` / `h5 a` / `.job-additional-information` | primary CSS selectors for the listing (keep the generic fallbacks after them) |
| `TheTatu13` / `turbomecanica-sa-nodejs-scraper` | the derived repo's owner / name |

Then adapt `parseListing` and `scrapeCareers` in `scraper/index.js` to the site,
and add a test per new cascade level.

## Testing

```bash
npm run test:unit          # always runs (placeholder-safe)
npm run test:integration   # self-skips until a company is configured + ANAF reachable
npm run test:e2e           # self-skips until configured; live careers-site + API otherwise
npm run test:consistency   # needs GITHUB_REPOSITORY + GITHUB_TOKEN (for a derived repo)
```

## License

MIT — see [LICENSE](LICENSE). Managed by
[ASOCIATIA OPORTUNITATI SI CARIERE](https://oportunitatisicariere.ro) for the
[peviitor.ro](https://peviitor.ro) job board.
