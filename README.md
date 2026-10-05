# AI Voice Companion — Call Server

Secure Cloudflare Worker backend for the [AI Voice Companion](https://muse.ai/s/ai-voice-companion-pxq62xqlbywv5) web app. It holds your Vapi API key server-side (never exposed to the browser) and triggers real outbound AI phone calls through [Vapi](https://vapi.ai).

## How it works

1. The web app collects: person's name, phone number, your name, and what the call is about — with confirm/edit and permission steps.
2. It `POST`s to this worker's `/call` endpoint.
3. The worker builds a fresh, personalized Vapi assistant per call (names + goal baked into the prompt) and dials the number.
4. The web app polls `GET /status?id=<callId>` for live call status.

## Deploy

```bash
npm i -g wrangler
wrangler login
wrangler secret put VAPI_API_KEY          # Vapi dashboard → API Keys (private key)
wrangler secret put VAPI_PHONE_NUMBER_ID  # Vapi dashboard → Phone Numbers → click number → copy ID
wrangler secret put CALL_SECRET            # invent any long random string
wrangler deploy
```

Or deploy without a terminal: Cloudflare dashboard → Workers → create worker → paste `vapi-call-worker.js` → Settings → Variables → add the three secrets → Deploy.

## Wire up the web app

Paste the worker URL (e.g. `https://mann-call-server.<you>.workers.dev`) and your `CALL_SECRET` into the web app's **Settings → Phone calling**.

## API

- `POST /call` — `{ to, personName, userName, goal, language }` → `{ callId, status }`
  - `to` must be E.164, e.g. `+91683889282`
  - Header: `x-call-secret: <CALL_SECRET>`
- `GET /status?id=<callId>` → `{ status, endedReason, summary, durationSeconds }`

## Notes

- Vapi gives $10 free credit on signup (no card). Real usage is ~$0.13–0.31/min all-in.
- Free Vapi numbers **cannot dial internationally** — upgrade the number or add a payment method to call non-US numbers.
- The per-call assistant is built inline (`assistant` object), so nothing needs pre-configuring in the Vapi dashboard. The prompt goes in `model.messages[]` (Vapi ignores `model.systemPrompt`).
