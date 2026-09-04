---
name: astronomy-api-observation-planning
description: Plan an observing session — get the positions of the Sun, Moon and planets for a location over a date range, and find the eclipses and apsides that fall inside it.
generated: '2026-09-04'
method: generated
source: openapi/astronomy-api-v3-openapi.yaml, openapi/astronomy-api-bodies-api-openapi.yml, openapi/astronomy-api-events-api-openapi.yml, https://docs.astronomyapi.com/
api: Astronomy API
version: v2 (production) / v3 (published reference draft)
operations:
  - getPositions        # v3 — GET /api/v3/positions
  - getEvents           # v3 — GET /api/v3/events
  - GET /bodies/positions        # v2 — production, no operationId declared
  - GET /bodies/positions/{body} # v2 — production, no operationId declared
  - GET /bodies/events/{body}    # v2 — production, no operationId declared
---

# Plan an observing session

Two calls: where things are, and what happens.

## Before you start

**Which version you are calling matters.** `/api/v2` is the surface that is live
today. `/api/v3` is a complete published contract that has **not shipped** — the
provider publishes it as a reference draft so the design can be argued with.
Call v2 unless you have confirmed v3 answers.

**Credentials.** Create an application in the dashboard; you get an Application
ID and an Application Secret. The secret is shown **once** and cannot be
retrieved — losing it means creating a new application and deleting the old one.

- v2: `Authorization: Basic base64(applicationId:applicationSecret)`
- v3: `Authorization: Bearer <application key>` — never in the query string

A wrong credential returns **403** on v2 (not 401). On v3 it returns 401.

## Step 1 — Positions

v2, all bodies at once. Ask for every body in one request rather than looping:
the provider's own docs say per-body loops are what gets callers throttled.

```
GET /api/v2/bodies/positions
  ?latitude=51.4779&longitude=-0.0015&elevation=0
  &from_date=2026-09-04&to_date=2026-09-11&time=22:00:00
  &output=rows
```

`latitude`, `longitude`, `from_date`, `to_date` and `time` are all **required**;
`elevation` is optional. `output` selects between two different response shapes —
`rows` gives `data.rows[].positions[]`, `table` gives `data.table.rows[].cells[]`.
Pick one and parse only that one.

**Hard bound:** the date range may not exceed **366 days** (added in changelog
2.14.0). Split anything longer into successive requests.

The v3 equivalent takes instants and a sampling step instead, so an altitude
curve is one request rather than twenty-four:

```
GET /api/v3/positions?bodies=moon&latitude=51.4779&longitude=-0.0015
  &from=2026-09-04T18:00:00Z&to=2026-09-05T06:00:00Z&step=PT15M&timezone=UTC
```

## Step 2 — Events

```
GET /api/v2/bodies/events/moon
  ?latitude=51.4779&longitude=-0.0015&elevation=0
  &from_date=2026-09-04&to_date=2026-12-31&time=00:00:00
```

v2 accepts **only `sun` and `moon`** here and reports eclipses. v3 accepts any
body and adds apsides (`type: apsis`), so if you need "when is Mars closest",
that answer does not exist on the production surface yet.

## Reading the numbers

Every v2 figure arrives as a **string**, including angles and distances, so
parse before you compare. Two v2 fields are known-wrong and are fixed in v3:

- `extraInfo.phase.angel` is a misspelling of `angle`.
- `extraInfo.phase.fraction` is scaled 0–0.067 **and runs backwards** — largest
  at new moon, zero at full. Do not read it as an illuminated fraction on v2.

`position.horizonal` (sic) is a duplicate of `position.horizontal`; ignore it.

Altitude below zero is a legitimate answer — the body is under the horizon.
Filter on `altitude > 0` if you only want what is up.

## Failures

| Status | Meaning | Do |
| --- | --- | --- |
| 403 | v2 auth failure — bad credential or bad base64 | Re-derive the string; do not retry unchanged |
| 422 | Validation failure — v2 returns a raw validator dump under `errors[]` | Read `errors[].argument`; fix the parameter |
| 429 | Rate limited on IP and overall consumption | Back off. **No `Retry-After` and no `RateLimit-*` header is served** — choose your own interval and use exponential backoff with jitter |
| 504 | Timeout | The provider states an identical retry succeeds |

There is no request-id header, so keep your own correlation id if you may need
to report a fault to contact@astronomyapi.com.

See `conventions/astronomy-api-conventions.yml` and
`errors/astronomy-api-problem-types.yml` in this repo.
