---
name: astronomy-api-deep-sky-search
description: Resolve a star or deep sky object by name, or find what sits at a sky position, and page through the results.
generated: '2026-09-04'
method: generated
source: openapi/astronomy-api-search-api-openapi.yml, openapi/astronomy-api-v3-openapi.yaml, https://docs.astronomyapi.com/endpoints/search, https://docs.astronomyapi.com/requests-and-response/body-properties-1
api: Astronomy API
version: v2 (production) / v3 (published reference draft)
operations:
  - search           # v3 — GET /api/v3/search
  - GET /search      # v2 — production, no operationId declared
---

# Search the catalogue

## By name

```
GET /api/v2/search?term=andromeda&match_type=fuzzy&limit=10&offset=0
Authorization: Basic base64(applicationId:applicationSecret)
```

- `term` is **required** on v2.
- `match_type` is `fuzzy` or `exact`. Use `exact` when you already hold a
  catalogue designation and `fuzzy` when you hold something a human typed.
- `order_by`, `limit`, `offset` control the page.

## By position

```
GET /api/v2/search?ra=0.712&dec=41.269
```

`ra` is in hours, `dec` in degrees. **On v3, `term` and the
`rightAscension`/`declination` pair are mutually exclusive** — send one or the
other, never both.

## Paging

v2 and v3 both page `/search` with `limit` and `offset`. Note v2 *declares*
`limit` and `offset` as strings in its contract even though they are numbers;
v3 declares them as integers. This is the only offset-paged endpoint —
`/positions` on v3 uses an opaque cursor instead (`meta.sampling.nextCursor`).

## Reading a result

Each item follows the shape documented at
<https://docs.astronomyapi.com/requests-and-response/body-properties-1>. In v3
it is a `CatalogueObject`: `id`, `name`, `type`, `subType`,
`crossIdentification`, `rightAscension`, `declination`. The `type`/`subType`
enumeration is at
<https://docs.astronomyapi.com/requests-and-response/dso-enums>.

v2 nests and stringifies the coordinates —
`position.equatorial.rightAscension.hours` as a string — where v3 returns
`rightAscension` as a number at the top level.

## Then what

Feed `rightAscension` and `declination` into
`astronomy-api-star-chart-image` with `view.type: "area"` to draw what you found.

## Failures

`403` on v2 for a bad credential, `422` for a validation failure with a raw
validator dump under `errors[]`, `429` when throttled with no `Retry-After` to
read. Nothing here writes, so every call is safe to retry.
