# CORS Tester

![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-f38020?logo=cloudflare&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178c6?logo=typescript&logoColor=white)
![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

> Inspect, debug, and fix Cross-Origin Resource Sharing (CORS) headers on any URL — a zero-dependency Cloudflare Worker with no client-side framework.

**Live:** [cors.infraforge.cc](https://cors.infraforge.cc)

## Features

- Sends a real HTTP request to a target URL with a chosen `Origin` header and inspects the response.
- Test any of `GET`, `POST`, `PUT`, `PATCH`, `HEAD`, `OPTIONS` — including preflight-style `OPTIONS` checks.
- Classifies the result at a glance: **not configured**, **wildcard**, **restricted (match)**, or **restricted (mismatch)**.
- Full response header table with CORS-relevant headers highlighted.
- Copyable/shareable link that reproduces the exact test (`?url=&origin=&method=`).
- Inline "how to fix" guidance when CORS headers are missing.

## How it works

The Worker performs the fetch server-side (not from the visitor's browser), so it always reflects what the target server actually sends — no need to open devtools or fight browser-enforced CORS to see the raw headers. Results are rendered into a single self-contained HTML response with no client-side JavaScript beyond a small "copy link" handler.

## Security

Since this Worker fetches arbitrary user-supplied URLs and renders response data back into HTML, it ships with a deliberately strict header set:

- **CSP** with a per-request nonce for both `script-src` and `style-src` — no `unsafe-inline` anywhere.
- `object-src 'none'`, `base-uri 'none'`, `form-action 'self'`, `frame-ancestors 'none'`, `upgrade-insecure-requests`.
- `Strict-Transport-Security` (HSTS) and `X-Frame-Options: DENY` (belt-and-suspenders with `frame-ancestors`).
- `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff`, `X-Permitted-Cross-Domain-Policies: none`.
- `Cross-Origin-Opener-Policy` / `Cross-Origin-Resource-Policy: same-origin`.
- `Permissions-Policy` disabling `geolocation`, `camera`, `microphone`, `payment`.
- `Cache-Control: no-store` — results are per-request and never cached.
- All user-controlled output is HTML-escaped before rendering.

## Tech stack

| Layer      | Tech                                    |
| ---------- | ---------------------------------------- |
| Runtime    | Cloudflare Workers (edge, no bindings)   |
| Language   | TypeScript (strict, ESM)                 |
| Styling    | Vanilla CSS, no build step               |
| Build/CLI  | Wrangler 4                               |
| Deploy     | Manual, via `wrangler deploy`            |

## Getting started

Requires **Node.js ≥ 22** and **pnpm**.

```bash
pnpm install
pnpm run dev       # wrangler dev → http://localhost:8787
```

## Scripts

| Command             | Description                                              |
| -------------------- | --------------------------------------------------------- |
| `pnpm run dev`        | Start the local dev server via `wrangler dev`.            |
| `pnpm run test`       | Type-check the project (`tsc --noEmit`) — no test suite.  |
| `pnpm run cf-typegen`  | Regenerate `Env`/binding types from `wrangler.toml`.       |
| `pnpm run deploy`     | Deploy via `wrangler deploy`.                             |

## Configuration

Key settings in [`wrangler.toml`](./wrangler.toml):

| Setting                | Value        | Notes                                              |
| ----------------------- | ------------ | --------------------------------------------------- |
| `compatibility_date`    | kept current | Bump periodically; see Cloudflare's changelog.       |
| `compatibility_flags`   | `nodejs_compat` | Enables Node.js built-ins.                        |
| `workers_dev`           | `false`      | Deploys to the custom domain configured in Cloudflare, not `*.workers.dev`. |
| `observability.enabled` | `true`       | Logs/traces enabled in the Cloudflare dashboard.     |

## Deploy

```bash
pnpm run deploy
```

Requires an authenticated Wrangler session (`wrangler login`) or a `CLOUDFLARE_API_TOKEN` in the environment. There is no CI/CD pipeline configured — deploys are manual.

## License

MIT
