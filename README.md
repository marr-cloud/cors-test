# CORS Tester

![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-f38020?logo=cloudflare&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178c6?logo=typescript&logoColor=white)
![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

> Inspect, debug, and fix Cross-Origin Resource Sharing (CORS) headers on any URL — a zero-dependency Cloudflare Worker with no client-side framework.

**Live:** [cors.maurrod.dev](https://cors.maurrod.dev)

## Features

- Sends a real HTTP request to a target URL with a chosen `Origin` header and inspects the response.
- Test any of `GET`, `POST`, `PUT`, `PATCH`, `HEAD`, `OPTIONS` — including preflight-style `OPTIONS` checks.
- Classifies the result at a glance: **not configured**, **wildcard**, **restricted (match)**, or **restricted (mismatch)**.
- Full response header table with CORS-relevant headers highlighted.
- Copyable/shareable link that reproduces the exact test (`/inspect?url=&origin=&method=`).
- Inline "how to fix" guidance when CORS headers are missing.

## How it works

The Worker performs the fetch server-side (not from the visitor's browser), so it always reflects what the target server actually sends — no need to open devtools or fight browser-enforced CORS to see the raw headers. Results are rendered into a single self-contained HTML response with no client-side JavaScript beyond a small "copy link" handler.

## Security

Since this Worker fetches arbitrary user-supplied URLs and renders response data back into HTML, it ships with a deliberately strict setup:

**Response headers** (on every response, including errors):

- **CSP** with `default-src 'none'` and a per-request nonce for `script-src` and `style-src` — no `unsafe-inline`, no third-party origins.
- **Trusted Types** enforced (`require-trusted-types-for 'script'; trusted-types 'none'`).
- `base-uri 'none'`, `form-action 'self'`, `frame-ancestors 'none'`, `upgrade-insecure-requests`.
- `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` and `X-Frame-Options: DENY`.
- Cross-origin isolation: `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Embedder-Policy: require-corp`, `Cross-Origin-Resource-Policy: same-origin`, `Origin-Agent-Cluster: ?1`.
- `Permissions-Policy` denying every powerful feature the page doesn't use.
- `Referrer-Policy: no-referrer`, `X-Content-Type-Options: nosniff`, `X-Permitted-Cross-Domain-Policies: none`.
- `Cache-Control: no-store, no-transform` — `no-transform` keeps Cloudflare from injecting its Web Analytics and JS Detections scripts; result pages also send `X-Robots-Tag: noindex, nofollow`.

**Outbound requests:**

- Redirects are reported, never followed (`redirect: "manual"`), with a 10 s timeout; the response body is discarded unread.
- Only `http(s)` URLs up to 2048 chars, no embedded credentials, and never the tool's own hostname.
- The `Origin` header is normalized to a bare origin, as a browser would send it.
- Per-IP rate limit: a zone WAF rate limiting rule on the `/inspect` path (hard limit), plus the Workers Rate Limiting binding (20 tests/minute, best-effort) as a second layer.

**Page:**

- Fonts are self-hosted (`public/fonts`, SIL Open Font License) — visiting the page contacts no third party.
- All user-controlled and target-supplied output is HTML-escaped before rendering.
- Only `GET`/`HEAD` on `/`, `/inspect` and `/robots.txt`; everything else is 404/405. Legacy `/?url=` links 301 to `/inspect`.
- `/.well-known/security.txt` and the HSTS header are managed zone-wide in Cloudflare (Security Center / Edge Certificates), shared by every Worker on `maurrod.dev`; the zone HSTS setting overrides the Worker's header, so keep them aligned.

## Tech stack

| Layer      | Tech                                    |
| ---------- | ---------------------------------------- |
| Runtime    | Cloudflare Workers + static assets, rate limiting binding |
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
