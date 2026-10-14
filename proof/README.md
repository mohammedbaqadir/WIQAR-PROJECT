# Proof — the third-party checks, kept raw

Run on 14 October 2026 against the live sites. Screenshots and machine reports live in this
folder; the summary with context is on the record page (`../index.html#proof`).

| File | What it is |
|---|---|
| `ssl-labs-before.png` · `ssl-labs-after.png` | Qualys SSL Labs reports (1:41pm and 3:06pm local). Grade **B in both** — capped by `*.workers.dev` still offering TLS 1.0/1.1 (Cloudflare's platform policy, not this project's). The after scan records **HSTS as present**; the before scan had none. |
| `lighthouse-mobile-before.json` · `lighthouse-mobile-after.json` · `lighthouse-mobile-after.html` · `lighthouse-mobile-after.png` | Google Lighthouse 13.5, mobile emulation. Accessibility **1.00**, best practices **0.96 → 1.00**, SEO **1.00**, agentic browsing (the AI-agent category) **1.00**; performance **0.66 → 0.59–0.66** across runs — the drag is total blocking time (~1.1 s of main-thread work on load). |
| `lighthouse-desktop-after.json` | Same run, desktop preset: all non-performance categories 1.00; performance 0.58, again TBT-bound. |

## Security headers, before → after (raw curl)

Before — storefront pages, `/catalog.json`, and the backend: **no security headers** (CORS only).

After — every response from all three:

```
strict-transport-security: max-age=31536000; includeSubDomains
content-security-policy:   pages: default-src 'self' + 8 inline-script SHA-256 hashes pinned
                           JSON routes: default-src 'none'; frame-ancestors 'none'
x-content-type-options:    nosniff
referrer-policy:           strict-origin-when-cross-origin
permissions-policy:        camera=(), microphone=(), geolocation=(), payment=(), usb=()
cross-origin-opener-policy: same-origin
x-frame-options:           DENY
```

Where they live: static pages get them from `public/_headers` (Workers static-assets metafile);
the catalogue route sets them in the Worker; the backend sets them once in its `json()` helper.
The CSP hashes pin the inline scripts Astro emits — if those scripts ever change, recompute
(documented in `_headers`).

## Notes on tooling

- **securityheaders.com** blocks automated scanners (its own bot protection) — the same values
  are visible in a human browser, and raw evidence is the curl output above.
- **PageSpeed Insights' anonymous API** was over quota during this run, so Lighthouse ran
  locally with Google's own engine (`bunx lighthouse@latest`, v13.5.0).
- Re-run anything: SSL Labs (`ssllabs.com/ssltest`), Lighthouse (`bunx lighthouse <url>`),
  headers (`curl -sI <url>`).
