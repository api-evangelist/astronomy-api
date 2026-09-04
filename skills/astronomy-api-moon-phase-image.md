---
name: astronomy-api-moon-phase-image
description: Render an image of the Moon for a location and date, and get back a URL — including choosing the orientation correctly for the caller's hemisphere.
generated: '2026-09-04'
method: generated
source: openapi/astronomy-api-studio-api-openapi.yml, openapi/astronomy-api-v3-openapi.yaml, https://docs.astronomyapi.com/endpoints/studio/moon-phase, https://docs.astronomyapi.com/known-issues
api: Astronomy API
version: v2 (production) / v3 (published reference draft)
operations:
  - createMoonPhase          # v3 — POST /api/v3/studio/moon-phase
  - POST /studio/moon-phase  # v2 — production, no operationId declared
---

# Render a moon phase image

One POST. It returns a URL, not an image.

```
POST /api/v2/studio/moon-phase
Authorization: Basic base64(applicationId:applicationSecret)
Content-Type: application/json

{
  "format": "png",
  "style": {
    "moonStyle": "default",
    "backgroundStyle": "stars",
    "backgroundColor": "red",
    "headingColor": "white",
    "textColor": "white"
  },
  "observer": { "latitude": 40.712772, "longitude": -73.935242, "date": "2026-09-04" },
  "view": { "type": "portrait-simple", "orientation": "north-up" }
}
```

Response: `{ "data": { "imageUrl": "..." } }`. Fetch that URL to get the image.

- `format` — `png` or `svg`.
- `view.type` — `portrait-simple` or `landscape-simple`.
- `view.orientation` — `north-up` or `south-up`.

## Get the orientation right

The provider lists this as a known issue: **which side of the Moon is up depends
on the observer's hemisphere.** If your caller is in the southern hemisphere and
you leave the default, the Moon will look upside down to them. Set
`view.orientation` from the sign of `observer.latitude` rather than accepting the
default.

## v3 differences

In v3, `time` moves out of the observer object and sits beside it, and
`observer.elevation` is accepted:

```json
{
  "observer": { "latitude": 40.712772, "longitude": -73.935242, "elevation": 0 },
  "time": "2026-09-04T22:00:00Z",
  "view": { "type": "portrait-simple", "orientation": "north-up" },
  "style": { "moonStyle": "default", "backgroundStyle": "stars" },
  "format": "png"
}
```

## Retries and repeats

This operation **creates nothing you need to undo**. It renders an image and
returns a URL; there is no resource in your account and no delete, cancel or
refund operation exists, because none is needed.

Identical requests return the **same cached URL without re-rendering**, so a
repeat is cheap and a retry is safe. There is no `Idempotency-Key` header on
this API and it does not need one.

On **504** — which the docs single out as most likely on the studio endpoints —
resend the identical request; the provider states it will work. On **429**, back
off; no `Retry-After` header is served.

## If you only need it in a web page

The provider ships a first-party widget that does this call and the DOM
insertion for you — see `components/astronomy-api-components.yml`. Note it puts
your base64 credential in page JavaScript, so scope the application's `Origin`
to your domain.
