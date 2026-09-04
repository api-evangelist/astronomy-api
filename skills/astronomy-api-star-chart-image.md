---
name: astronomy-api-star-chart-image
description: Render a star chart for an observer — framed either on a named constellation or on a point in the sky with a zoom level — and get back an image URL.
generated: '2026-09-04'
method: generated
source: openapi/astronomy-api-studio-api-openapi.yml, openapi/astronomy-api-v3-openapi.yaml, https://docs.astronomyapi.com/endpoints/studio/star-chart
api: Astronomy API
version: v2 (production) / v3 (published reference draft)
operations:
  - createStarChart          # v3 — POST /api/v3/studio/star-chart
  - POST /studio/star-chart  # v2 — production, no operationId declared
---

# Render a star chart

Two framings, one endpoint. Choose the framing with `view.type`.

## Framed on a constellation

```
POST /api/v2/studio/star-chart
Authorization: Basic base64(applicationId:applicationSecret)
Content-Type: application/json

{
  "style": "navy",
  "observer": { "latitude": 12.775867, "longitude": -23.39733, "date": "2026-09-04" },
  "view": { "type": "constellation", "parameters": { "constellation": "ori" } }
}
```

`constellation` is a **three-letter IAU id** — `ori`, `and`, `leo`. The complete
list is at
<https://docs.astronomyapi.com/requests-and-response/constellation-enums>.
Do not guess one; a wrong id is a 422.

## Framed on a sky position

```json
{
  "style": "default",
  "observer": { "latitude": 51.4779, "longitude": -0.0015, "date": "2026-09-04" },
  "view": {
    "type": "area",
    "parameters": {
      "position": { "equatorial": { "rightAscension": 5.5, "declination": -5.4 } },
      "zoom": 4
    }
  }
}
```

Right ascension is in **hours**; declination is in **degrees**. In v3 the
`equatorial` wrapper is dropped — the position is `{ "rightAscension": 5.5,
"declination": -5.4 }` — because only one frame is ever involved.

`style` is one of `default`, `inverted`, `navy`, `red`.

Response: `{ "data": { "imageUrl": "..." } }`.

## Finding something to point at

If you have a name rather than coordinates, resolve it first with
`astronomy-api-deep-sky-search`, then pass the returned right ascension and
declination into the `area` view.

## Retries

Renders only — nothing is created that could need reversing, and an identical
request returns the same cached URL. Retry an identical request on **504**; back
off on **429** without a `Retry-After` to guide you.
