# Beans and Byte — voice-to-action ordering prototype

A single-page prototype that shows a voice layer taking real actions on a web page:
hold the mic, speak naturally, and the order form fills itself in.

**No build step, no backend.** One file: `index.html`. Type is Inter, loaded from
Google Fonts. Voice works out of the box with the browser's built-in recognizer; an
optional free Gemini API key upgrades accuracy a lot.

## Design

The UI follows **Apple's Human Interface Guidelines**, the closest fit for a small,
single-purpose utility: one centred card, a neutral grey ramp (`#f5f5f7` page,
`#ffffff` surface, `#1d1d1f` / `#6e6e73` / `#86868b` labels), hairline separators,
and system blue `#0071e3` for the single primary action.

- **Segmented controls** use the iOS pattern: a grey track with a white raised pill
  for the selected option (single-select by construction).
- **Text fields** are quiet grey fills that turn white with a blue focus ring.
- **The voice bar** is a translucent glass pill pinned to the bottom, so the form
  stays the only thing in the centre of the screen.
- Type scale is tight (13/15/19/24 px, negative tracking on headings), and all
  tokens live in `:root` — changing the accent is a one-line edit.

## Run it

Voice input needs **Chrome or Edge**, and a **secure context** (`http://localhost` or HTTPS).

```powershell
cd C:\Users\impla\development\projects\vta
npx serve --listen 4173
```

Open **http://localhost:4173** — and keep using that exact URL.

> Why the fixed port? Chrome stores microphone permission **per origin**
> (`scheme + host + port`). A new port is a new site, so Chrome asks again.
> `file://` pages also re-prompt (or block) and are not reliable for mic access.

## First run

1. Press the mic button once → Chrome asks for the microphone → **Allow**.
   (Dismissing the prompt with ✕ makes Chrome keep asking.)
2. The button turns blue and the status reads **"Mic ready — hold to talk"**.
3. Hold the mic (or hold **Space**) → speak → release.

If it still asks every time:

- Open `chrome://settings/content/microphone` and make sure `localhost:4173` is **Allowed**.
- Don't test inside an embedded preview pane — open the URL in a normal Chrome/Edge window.
- Use `http://localhost:...`, not `http://192.168.x.x:...` (not a secure context).

## Demo script

> Hold the mic → *"My name is Alex, email alex at example dot com, two coffees, paying by card"*
> → release → five fields fill and flash, total becomes **$7.00**
> → *"place the order"* → confirmation with an order number.

Also understood: *"start over"*, *"I'm done"*, *"one tea"*, *"three hot chocolates"*, *"paying cash"*.

## Accents

The **Accent** setting (gear icon in the voice bar) defaults to **English (India)**,
because Chrome's `en-IN` model handles Indian English far better than `en-US`. It's
remembered in `localStorage`; switch to US/UK English for other voices.

Speech recognition is never perfect, so the parser is accent-tolerant:

- it requests **5 alternatives** from Chrome and picks the hearing that maps to the most fields
- tokens are matched **phonetically (Soundex)** and by edit distance, so
  "copy" → *coffee*, "tree" → *three*, "tee" → *tea*, "kash" → *cash*
- ambiguous words need context: "I want **to** order" does not become amount 2

If recognition still struggles, use the **type-instead** box (same pipeline, no mic).

## Better accuracy: Gemini transcription (optional, free)

Chrome's recognizer is the weak link on accented speech. The page can instead send
the recorded audio to Google's speech models on the Gemini API **free tier** (no card).
To enable it:

1. Get a key at <https://aistudio.google.com/apikey>.
2. In the voice bar, open the **gear icon**, paste the key, press **Use**.
3. Hold the mic as usual. The status shows *Transcribing…* instead of *Listening…*.

### Model fallback

Free-tier access varies per project, so the app tries models in order and sticks
with the first one that works:

| Order | Model | Free-tier limits (typical) | Role |
|---|---|---|---|
| 1 | `gemini-3.5-transcribe` | often not available on free tier (503) | purpose-built ASR, best accuracy |
| 2 | `gemini-3.5-flash-lite` | 15 RPM / 500 RPD | audio understanding + strict "transcribe only" prompt |

If the first model answers *"experiencing high demand"* (a 503 that usually means
"not enabled for your project"), the app logs `… failed — trying fallback` and
retries with Flash Lite. A bad key is treated as fatal: one request, then
`Gemini: key invalid — paste it again`.

Notes:

- The key is stored in this browser's `localStorage` only; it never leaves the machine
  except in the request to `generativelanguage.googleapis.com`.
- Requests are direct from the page (CORS is open), so there is **no backend**. That also
  means the key is visible in devtools — fine for a local demo, not for production.
- Free tier is free of charge but Google may use the audio to improve their products.
  Don't send anything sensitive.
- `Api-Revision` must **not** be sent: that header breaks the CORS preflight.
- Press **Off** to go back to the built-in recognizer.

## How it works

```
mic → Web Speech API (free, browser-native)   ← default
    → or MediaRecorder → Gemini 3.5 Transcribe ← optional key
    → rules parser (deterministic, can't fail on demo vocabulary)
    → DOM actions (set fields, press segmented buttons, submit)
    → status pill + action log + field flash
```

The parser only writes values that exist on the page and are valid
(order type ∈ menu, amount ∈ {1,2,3}, email shape checked). No LLM is used for
the action mapping — that stays deterministic on purpose.

### Known limits (v0, on purpose)

- Chrome's built-in recognizer needs a network connection (Chrome's speech service).
- Safari / Firefox: mic button disables gracefully; the form still works by hand.
- Free-form phrasing outside the demo vocabulary won't map — that's the v2 (LLM) story.
- Built-in recognizer defaults to `en-IN`; switch the Accent dropdown for other voices.
