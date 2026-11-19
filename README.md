# Bean & Byte — voice-to-action ordering prototype

A single-page prototype that shows a voice layer taking real actions on a web page:
hold the mic, speak naturally, and the order form fills itself in.

**Zero dependencies, zero backend, zero API keys.** One file: `index.html`.

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
2. The button turns orange and the status reads **"Mic ready — hold to talk"**.
3. Hold the mic (or hold **Space**) → speak → release.

If it still asks every time:

- Open `chrome://settings/content/microphone` and make sure `localhost:4173` is **Allowed**.
- Don't test inside an embedded preview pane — open the URL in a normal Chrome/Edge window.
- Use `http://localhost:...`, not `http://192.168.x.x:...` (not a secure context).

## Demo script

> Hold the mic → *"My name is Alex, email alex at example dot com, two coffees, paying by card"*
> → release → four fields fill and flash, total becomes **$7.00**
> → *"place the order"* → confirmation with an order number.

Also understood: *"start over"*, *"I'm done"*, *"one tea"*, *"three hot chocolates"*, *"paying cash"*.

## How it works

```
mic → Web Speech API (free, browser-native)
    → rules parser (deterministic, can't fail on demo vocabulary)
    → DOM actions (set fields, press segmented buttons, submit)
    → status pill + action log + field flash
```

The parser only writes values that exist on the page and are valid
(order type ∈ menu, amount ∈ {1,2,3}, email shape checked). No LLM, no network
beyond Chrome's speech service.

### Known limits (v0, on purpose)

- Voice recognition needs a network connection (Chrome's speech service).
- Safari / Firefox: mic button disables gracefully; the form still works by hand.
- Free-form phrasing outside the demo vocabulary won't map — that's the v2 (LLM) story.
- `en-US` recognition only.
