# Deploying TravelSense on Cloudflare Pages

Same flat file set as before. Cloudflare runs `_worker.js` for every request:
`/api/*` is handled by the worker, everything else is served as a static file.

## Files

All at the **root** of the repository. No folders.

| File | Purpose |
|---|---|
| `index.html` | The whole app |
| `_worker.js` | API: chat, speech, signup capture |
| `health.txt` | Diagnostic |
| `DEPLOY-CLOUDFLARE.md` | This guide |
| `netlify.toml`, `chat.js`, `hello.js` | Only needed if you move back to Netlify. Harmless on Cloudflare. |

## Steps

1. **Cloudflare dashboard** → Workers & Pages → Create → **Pages** →
   Connect to Git → choose your repository.

2. Build settings:
   - Framework preset: **None**
   - Build command: **leave empty**
   - Build output directory: **`/`** (a single forward slash)

3. **Settings → Environment variables → Production**:

   | Variable | Required | Purpose |
   |---|---|---|
   | `ANTHROPIC_API_KEY` | yes | The conversation and recommendation text |
   | `GOOGLE_TTS_API_KEY` | no | Natural voice; without it the device voice is used |
   | `GOOGLE_TTS_VOICE` | no | Defaults to `en-US-Chirp3-HD-Achernar` |
   | `GOOGLE_TTS_STYLE` | no | The spoken style instruction |
   | `GA4_MEASUREMENT_ID` | no | Server-side cost tracking |
   | `GA4_API_SECRET` | no | Server-side cost tracking |

4. **Redeploy.** Cloudflare does not apply new variables to an existing deploy.

## Verify

| URL | Expected |
|---|---|
| `/health.txt` | plain text — static files are being served |
| `/api/hello` | JSON showing which keys are detected (never the keys themselves) |
| `/api/chat` | **405** — 405 means live. 404 means `_worker.js` isn't at the root |
| `/` | the app |

## Logs

Workers & Pages → your project → Deployments → **Functions**, or stream them:

```bash
npx wrangler pages deployment tail
```

- `ts_metrics` — tokens, cost, latency, retries, TTS usage
- `ts_signup` — beta signups with email, name and source

## Notes

- The app auto-detects its host, so the same `index.html` works on Cloudflare
  and Netlify. On a custom domain it defaults to `/api/chat`; if you move back
  to Netlify with a custom domain, set `window.__TS_API_ENDPOINT` near the top
  of `index.html` to `/.netlify/functions/chat`.
- Free tier: 100,000 worker requests/day. A full search is roughly 10–15
  requests, so about 6,000 searches a day.
- Logs are not permanent. Signup emails should be copied out periodically.
