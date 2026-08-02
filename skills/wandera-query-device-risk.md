---
name: Query and override device risk (Wandera RADAR)
description: Authenticate to the Wandera / Jamf Security Cloud RADAR Risk API and list or override the risk state of enrolled devices.
api: openapi/wandera-risk-api-openapi.yml
operations:
  - login
  - getDevices
  - overrideDeviceRisk
---

# Query and override device risk (Wandera RADAR)

Operating instructions for using the Wandera (Jamf Security Cloud / RADAR) Risk API
correctly. Base host: `https://api.wandera.com`.

## 1. Authenticate (`login`)

- Create an Application ID and Application Secret in **Security Integrations** under
  RADAR Settings.
- `POST /v1/login` with those credentials as HTTP **Basic** authorization.
- The response `token` is a **JWT valid for 15 minutes**. Cache it and refresh before
  expiry; do not request a new token on every call.
- Present the token as `Authorization: Bearer <token>` on all other calls.

## 2. List device risk states (`getDevices`)

- `GET /risk/v1/devices?page=<n>&pageSize=<=100`.
- Page through results: `pageSize` **must not exceed 100** (else `MAX_LIMIT_PER_PAGE`,
  400) and `page`/`pageSize` must not be negative (`NEGATIVE_PAGE`, `NEGATIVE_PAGE_SIZE`).
- Each device record carries `deviceId`, `risk` (LOW/MEDIUM/HIGH), OS type/version,
  and location.

## 3. Override device risk (`overrideDeviceRisk`)

- `PUT /risk/v1/override` with body `{ "risk": "MEDIUM", "source": "WANDERA",
  "deviceIds": ["<uuid>", ...] }`.
- The operation is idempotent by HTTP semantics — re-applying the same risk to the same
  device IDs yields the same state.

## Conventions and error handling

- **Rate limits:** 5 requests/second and 10,000 requests/day per integration. On `429`
  (`TOO_MANY_REQUESTS`), read `X-Rate-Limit-Retry-After-Milliseconds` and back off.
- **Errors** use the envelope `{ logref, message, error, statusCode }`. Branch on the
  stable `logref` code, not the human `message`.
- `403 Forbidden` usually means an expired (>15 min) or invalid JWT — re-authenticate.
