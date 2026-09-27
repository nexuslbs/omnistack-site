# OmniStack POC site

Static, public proof-of-concept pages for two of the three built MVPs of the
`mvp` kanban task `task_mvp_mvp_setup_to_earning_all_3_cases_p1_p2_p3`.

This repository is served by GitHub Pages at
<https://nexuslbs.github.io/omnistack-site/>. It contains **POC pages only** -
no production service, no live backend, no live payment.

## What is served

| Path | Content |
|---|---|
| `/index.html` | Plain landing page linking to `/p3/` and `/p1/`. |
| `/p3/` | The P3 **Brief Studio** published static bundle: the delivered-briefs index plus three self-contained delivered briefs (`37signals/`, `demo-fixture/`, `pipo-saude/`), each with `brief.html`, `invoice.html`, `meta.json` and `data-pack.json`. |
| `/p1/` | The P1 micro-asset **cron-explain** page (`index.html`), its contract (`openapi.json`), its manifest (`asset.yml`) and its upstream `README.md`. |

## Where the content came from (reused verbatim, never rebuilt or re-rendered)

Upstream repository: `nexuslbs/mvp` (<https://github.com/nexuslbs/mvp>).

* **P3 Brief Studio** - source tree `/opt/workspace/tmp/p3-mvp/mvp/brief-studio/site/published/`,
  from upstream HEAD `927ada5a33ed0d6b31d6921b814ae53972cb4f38` at copy time.
  The `published/` directory is git-ignored runtime output upstream, so it is not
  tracked at that commit; the copy here is the on-disk bundle as produced by the
  Brief Studio build. Key page:
  `p3/demo-fixture/brief.html` (14445 bytes,
  sha256 `bed4bdb959ec7321d31646c91363248d612af4ab5cdf19b3d5fa25c2705ffe51`).
* **P1 cron-explain** - source tree
  `/opt/workspace/tmp/mvp-p1/mvp/asset-pipeline/assets/cron-explain/`,
  from upstream HEAD `c411fa2475d145bf6a2572e12e13a1a22a2ab86b`.
  `p1/index.html` is the upstream page plus a clearly marked POC hosting note;
  `README.md`, `openapi.json` and `asset.yml` are copies.

## Honest limitations (POC)

* **P1 API is not deployed here.** GitHub Pages is static hosting only. The
  `cron-explain` page's form posts to `/api/explain`; there is no such backend
  at this origin, so the form returns an error. The asset runs locally exactly
  as its `README.md` documents (`scripts/run_local.sh start`, port 8099). No
  public API URL is claimed.
* **P3 payment links are local POC mocks.** The delivered-brief index currently
  links checkout buttons at `http://127.0.0.1:8799/pay/...`, which only resolve
  on the build machine. They are not live payment links. No revenue is claimed
  (`metrics.json` `revenue_usd_30d` is `0.0`).
* The pages are unlisted proof-of-concept artifacts, not a launched product.

## Branches

`main` and `gh-pages` carry the same tree. The `gh-pages` branch exists so the
repository owner - or GitHub itself, when the `gh-pages` branch convention
applies - can serve this tree with GitHub Pages without requiring the
`pages` API permission on the deployment App.
