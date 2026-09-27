# cron-explain

A stateless public JSON API that turns a cron expression into a plain-language
description and its next five run times. Built for developers who need a
schedule check without reading `man 5 crontab`.

One line hypothesis: "Backend and DevOps developers who paste a cron expression
into an API want a plain-language description and the next five run times, with
no account, no stored input, and no key."

* Class: `micro_api` (see `asset.yml`).
* Runtime: Python 3 standard library only, no pip installs.
* Cost: 0 USD on the Cloudflare Workers free tier, `*.workers.dev`, no custom domain.
* Privacy: no account, no cookies, no `localStorage`, and no raw IP or User-Agent
  is stored. The request body is parsed in memory and never written to disk.

## Public routes

| Route | Method | Returns |
|---|---|---|
| `/` | GET | HTML page, `Content-Type: text/html; charset=utf-8` |
| `/api/explain` | POST | `{"expr","tz","valid","description","next_runs"}` or HTTP 400 |
| `/api/health` | GET | `{"status":"ok"}` |
| `/api/stats` | GET | `{"total_calls","api_calls_7d","pageviews_7d","unique_visitors_7d"}` |

Copy-paste examples (against the local service on port 8099):

```sh
curl -s http://127.0.0.1:8099/api/health
# {"status":"ok"}

curl -s -X POST http://127.0.0.1:8099/api/explain \
  -H 'Content-Type: application/json' \
  -d '{"expr":"*/5 9-17 * * 1-5","tz":"UTC","count":5}'
# {"expr":"*/5 9-17 * * 1-5","tz":"UTC","valid":true,"description":"Every 5 minutes, between 09:00 and 17:59, Monday through Friday","next_runs":["2026-09-28T09:00:00+00:00","2026-09-28T09:05:00+00:00","2026-09-28T09:10:00+00:00","2026-09-28T09:15:00+00:00","2026-09-28T09:20:00+00:00"]}

curl -s -X POST http://127.0.0.1:8099/api/explain \
  -H 'Content-Type: application/json' -d '{"expr":"@daily","tz":"America/New_York"}'

curl -s -X POST http://127.0.0.1:8099/api/explain \
  -H 'Content-Type: application/json' -d '{"expr":"99 99 * * *","tz":"UTC"}'
# {"error":"invalid_cron","message":"minute value out of range 0-59: '99'","expr":"99 99 * * *"}

curl -s http://127.0.0.1:8099/api/stats
```

Supported expressions: standard 5-field cron (minute hour day-of-month month
day-of-week) plus `@hourly`, `@daily`, `@weekly`, `@monthly`. Steps, ranges and
lists are supported (`*/5`, `9-17`, `1,3,5`), and Sunday accepts both `0` and
`7`. `tz` is any IANA name resolved with the standard-library `zoneinfo`;
`count` defaults to 5 and is capped at 50.

## Run locally

```sh
./scripts/run_local.sh start    # python3 app.py --host 127.0.0.1 --port 8099, nohup, pidfile /tmp/cron-explain.pid
./scripts/acceptance.sh         # the 5 acceptance checks, prints PASS-A1 .. PASS-A5
python3 test_app.py             # HTTP and PII tests
python3 test_cron.py            # parser and next-run determinism tests
./scripts/run_local.sh stop     # kill the process from /tmp/cron-explain.pid
```

`run_local.sh start` waits for `/api/health` and then resets `traffic.jsonl` so
the next measurement window starts at zero. A plain background process is used;
this asset ships no Dockerfile and no compose file.

`traffic.jsonl` is the source the `local_jsonl` analytics connector reads. It
is git-ignored because it is a runtime output; `traffic.sample.jsonl` documents
the row format with 5 rows. Each row has exactly
`ts, path, method, status, ip_hash, ua_hash`. The IP and the User-Agent are
hashed with a per-process random salt and truncated to 16 hex chars, the query
string is stripped, and the body is never a logging input.

Every request appends its row before the server builds and writes the response,
and the file is closed before the response is sent. As a result `GET /api/stats`
counts its own request: read `traffic.jsonl` immediately after the stats
response and `total_calls` equals the number of rows, while `api_calls_7d`
counts every `/api/*` row, including the stats request itself.

## Deploy to the Cloudflare Workers free tier

The Workers bundle is self-contained in `worker/`:

* `worker/worker.js`: a plain module worker (`export default { fetch }`) with
  the same routes and compact JSON contract. Counters are in-memory; the
  `ANALYTICS` Workers Analytics Engine binding is optional and token-gated and
  is not enabled in this bundle.
* `wrangler.toml` (this directory): the canonical config the pipeline's
  `cloudflare` connector reads. `python3 -m assetpipeline publish --slug
  cron-explain --connector cloudflare` runs `npx wrangler deploy` from this
  directory, so wrangler discovers this file. It sets `name = "cron-explain"`,
  `main = "worker/worker.js"`, `workers_dev = true`, no paid features and no
  custom domain.
* `worker/wrangler.toml`: the config for the direct wrangler path used by
  `scripts/deploy_cloudflare.sh`. It is identical except `main = "worker.js"`,
  because wrangler resolves `main` relative to the config file's directory; so
  `npx wrangler deploy --config worker/wrangler.toml` keeps working.

Deploy for real through the pipeline (token-gated in `assetpipeline`):

```sh
cd ../.. && python3 -m assetpipeline publish --slug cron-explain --connector cloudflare
```

Deploy for real through the direct wrangler script:

```sh
./scripts/deploy_cloudflare.sh
```

With `$CLOUDFLARE_API_TOKEN` unset the script prints the exact human handover and
exits 2. This is the raw handover:

```
HUMAN HANDOVER REQUIRED: create a Cloudflare API token with the "Edit Cloudflare Workers" template at https://dash.cloudflare.com/profile/api-tokens.
Set it in the environment variable CLOUDFLARE_API_TOKEN (platform secrets store; never commit it).
Then run: npx wrangler deploy --config worker/wrangler.toml
```

With the token set, the script runs
`npx wrangler deploy --config worker/wrangler.toml` and echoes `DEPLOYED_URL=...`.
It never fakes a deploy. The intended public URL shape is
`https://cron-explain.SUBDOMAIN.workers.dev` (`deploy.url` in `asset.yml`);
`deploy.endpoint` stays `null` until the real `publish` fills it.

## Cost

0 USD. Workers free tier: 100,000 requests/day and 10 ms CPU per request. No
custom domain, no payment method, no KV, no D1, no Analytics Engine binding.
The `limits` caps in `asset.yml` are `usd_max: 5.00`, `tokens_max: 400000`,
`human_minutes_max: 180`, and `publish_requires_approval: true`.

## Kill rule and measurement

Pre-registered in `asset.yml` before the first measurement:

| Field | Value |
|---|---|
| `kill_rule.metric` | `api_calls_7d` |
| `kill_rule.operator` | `<` |
| `kill_rule.threshold` | `25` |
| `kill_rule.grace_days` | `30` |
| `kill_rule.escalate_after_days` | `120` |
| `measurement.source` | `cloudflare` (the weekly public reading) |
| `measurement.kind` | `api_calls_7d` |

In words: after the 30-day grace clock, if `api_calls_7d` is below 25 the asset
is killed; at 120 days with traffic but no revenue it escalates. `measure`
writes the four standard fields (`pageviews_7d`, `unique_visitors_7d`,
`api_calls_7d`, `revenue_usd_30d`) to `metrics.json`, and the local
`local_jsonl` connector counts `/api/*` requests from `traffic.jsonl` as
`api_calls_7d`.

## Layout

```
app.py                 stdlib server, cron parser, PII-free traffic log
index.html             the page served at /
test_app.py            HTTP + PII tests
test_cron.py           cron parser + next-run tests
asset.yml              the pipeline manifest
openapi.json           the four routes
scripts/run_local.sh   start / stop / status
scripts/acceptance.sh  the 5 acceptance checks
scripts/deploy_cloudflare.sh  token-gated wrangler deploy
worker/worker.js       Cloudflare Workers bundle
wrangler.toml          canonical pipeline connector config (main=worker/worker.js)
worker/wrangler.toml   direct wrangler config (main=worker.js)
traffic.sample.jsonl   5-row sample of the traffic log format
```
