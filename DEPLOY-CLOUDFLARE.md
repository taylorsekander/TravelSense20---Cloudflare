# Deploying TravelSense on Cloudflare

Your project was created as a **Worker** (Cloudflare's current default for Git
projects), which runs `npx wrangler deploy`. These files are set up for that.

## Upload exactly these six

All at the **root** of the repository. No folders.

| File | Purpose |
|---|---|
| `index.html` | The whole app |
| `_worker.js` | The API: chat, speech, signup capture |
| `wrangler.json` | Tells Cloudflare which file is the worker |
| `.assetsignore` | Stops `_worker.js` being published publicly |
| `health.txt` | Diagnostic — delete once the site works |
| `DEPLOY-CLOUDFLARE.md` | This guide |

**Delete everything else from the repository.** Your last build read 67 files
and published all of them. Every static file in the repo is served to anyone
who requests it.

Files to remove if present: `netlify.toml`, `chat.js`, `hello.js`, `DEPLOY.md`,
any `index (1).html` style duplicates, and anything left over from earlier
attempts.

Note: `.assetsignore` and `wrangler.json` start with a dot or are config —
GitHub's web uploader handles both fine, but confirm `.assetsignore` kept its
leading dot after upload.

## Why the build failed

Wrangler was about to publish `_worker.js` as a public static file. That would
have exposed your server-side code. `.assetsignore` excludes it, and
`wrangler.json` tells Cloudflare to run it as the worker instead.

## Environment variables

Dashboard → your Worker → **Settings → Variables and Secrets** → Production:

| Variable | Required | Purpose |
|---|---|---|
| `ANTHROPIC_API_KEY` | yes | Conversation and recommendation text |
| `GOOGLE_TTS_API_KEY` | no | Natural voice; falls back to the device voice |
| `GOOGLE_TTS_VOICE` | no | Defaults to `en-US-Chirp3-HD-Achernar` |
| `GOOGLE_TTS_STYLE` | no | Spoken style instruction |
| `GA4_MEASUREMENT_ID` | no | Server-side cost tracking |
| `GA4_API_SECRET` | no | Server-side cost tracking |

Use **Secret** rather than plain text for the API keys. Redeploy afterwards —
new variables are not applied to an existing deploy.

## Verify

| URL | Expected |
|---|---|
| `/health.txt` | plain text — static files are serving |
| `/api/hello` | JSON listing which keys are detected, never the keys themselves |
| `/api/chat` | **405** — 405 means live. 404 means the worker is not running |
| `/` | the app |

Also confirm `/_worker.js` returns **404**. If it returns JavaScript, your
server code is public and `.assetsignore` is not being applied.

## Logs

Dashboard → your Worker → **Logs**, or stream them:

```bash
npx wrangler tail
```

- `ts_metrics` — tokens, cost, latency, retries, speech usage
- `ts_signup` — beta signups with email, name and source

## Notes

- If `wrangler.json` reports a name mismatch, change `"name"` to match your
  Worker's name in the dashboard.
- Free tier: 100,000 worker requests/day. A search is roughly 10–15 requests.
- Logs are not permanent. Copy signup emails out periodically.
