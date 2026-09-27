# OmniStack POC site

Static, public proof-of-concept pages for two of the three built MVPs of the
`mvp` kanban task `task_mvp_mvp_setup_to_earning_all_3_cases_p1_p2_p3`.

This repository is served by **Cloudflare Pages** at
<https://omnistack-site.pages.dev/>. It contains **POC pages only** - no
production service, no live backend, and **no live payment link**. Nothing here
can take money.

## What is served

| Path | Content |
|---|---|
| `/index.html` | Plain landing page linking to `/p3/` and `/p1/`. |
| `/p3/` | The P3 **Brief Studio** published static bundle: the delivered-briefs index plus three self-contained delivered briefs (`37signals/`, `demo-fixture/`, `pipo-saude/`), each with `brief.html`, `invoice.html`, `meta.json` and `data-pack.json`. |
| `/p1/` | The P1 micro-asset **cron-explain** page (`index.html`), its contract (`openapi.json`), its manifest (`asset.yml`) and its upstream `README.md`. |
| `/_redirects` | Cloudflare Pages proxy rules that keep the canonical `.html` URLs answering `200` instead of the Pages default `308` to the extensionless path. |

## Where the content came from

Upstream repository: `nexuslbs/mvp` (<https://github.com/nexuslbs/mvp>).

* **P3 Brief Studio** - source tree `brief-studio/site/published/` (git-ignored
  runtime output upstream), rebuilt with the project's own CLI
  (`python3 -m briefstudio build` + `deliver --checkout test`) after the renderer
  was changed to mark the local test checkout as an explicit sandbox. The
  `published/` directory is not tracked, so the copy here is the on-disk bundle.
* **P1 cron-explain** - source tree `asset-pipeline/assets/cron-explain/`.
  `p1/index.html` is the upstream page plus a clearly marked POC hosting note;
  `README.md`, `openapi.json` and `asset.yml` are copies.

## Honest limitations (POC)

* **P1 API is not deployed here.** This is static hosting only. The
  `cron-explain` page's form posts to `/api/explain`; there is no such backend
  at this origin, so the form returns an error. The asset runs locally exactly
  as its `README.md` documents (`scripts/run_local.sh start`, port 8099). No
  public API URL is claimed.
* **P3 has NO live checkout.** The delivered briefs were produced through
  brief-studio's local test mock (`provider: test`), which a stranger cannot
  reach. The pages render that as an explicit
  `SANDBOX PLACEHOLDER - no real payment is possible from this page`; the
  loopback mock URL is not rendered as a link. A real provider link appears
  automatically only once a merchant-of-record key (`PADDLE_API_KEY` +
  `PADDLE_PRICE_ID`, `STRIPE_SECRET_KEY`, ...) exists and `deliver` is re-run.
  No revenue is claimed (`metrics.json` `revenue_usd_30d` is `0.0`).
* The pages are unlisted proof-of-concept artifacts, not a launched product.

## Branches

`main` and `gh-pages` carry the same tree. The `gh-pages` branch exists so the
repository owner - or GitHub itself, when the `gh-pages` branch convention
applies - can serve this tree with GitHub Pages without requiring the
`pages` API permission on the deployment App.
