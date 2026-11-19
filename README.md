# Beans & Byte

**Order a coffee by talking to the page.**

Beans & Byte is a small web page with an order form — name, email, drink, quantity,
payment. Instead of typing, you hold the microphone button and say your order out loud.
The form fills itself in, the total updates, and you can finish by saying *"place the
order."*

No app to install. It is one web page.

---

## What it does

Say something like:

> *"My name is Alex, email alex at example dot com, two coffees, paying by card."*

and the page will:

- write **Alex** in the name field
- write **alex@example.com** in the email field
- select **Coffee**
- select **2**
- select **Card**
- change the total to **$7.00**

Each field that changes gives a short blue flash, so you can see what was understood.
You can also say *"place the order"* to finish, or *"start over"* to clear the form.

You can still use the form by hand — clicking and typing work normally.

---

## How to run it

You need **Chrome or Edge** and a microphone.

```powershell
cd C:\Users\impla\development\projects\vta
npx serve --listen 4173
```

Then open **http://localhost:4173** in a normal browser window.

1. Click the mic button once. The browser asks for microphone access — choose **Allow**.
2. Hold the mic button (or hold the **Space** key) and speak.
3. Let go when you are done. The form updates.

Keep using the same address (`localhost:4173`). The browser remembers microphone
permission per address, so a different port means it asks again.

> If the mic is blocked, a small text box appears in the voice bar. You can type the
> same sentence there and the page will handle it exactly the same way.

---

## How it works

In plain terms, three steps:

1. **Listen.** While you hold the button, the browser turns your speech into text.
   This is built into Chrome and Edge — no account, no extra service.
2. **Understand.** The page reads that text and picks out the useful parts: a name,
   an email, a drink, a number, a payment method. It only accepts values that exist on
   the page (the three drinks, quantities 1–3, card or cash), so it cannot put nonsense
   into the form.
3. **Act.** It fills those fields, highlights them, recalculates the total, and can
   even press "Place order" for you.

That is the whole idea: **speech goes in, real actions happen on the page.** It is not a
chatbot, and it does not guess — if it hears nothing useful, it says so and leaves the
form alone.

The page also handles accents and common mishearings. "Copy" becomes *coffee*, "tree"
becomes *three*, "kash" becomes *cash*. The Accent setting (gear icon) is set to
**English (India)** by default and can be switched to US or UK English.

---

## Who this helps

A form looks simple, but it is not simple for everyone. Voice ordering helps people who:

- **find typing difficult** — because of arthritis, a tremor, an injury, or a repetitive
  strain injury
- **cannot see the screen well** — they can speak an order without hunting for small
  fields and buttons
- **are using a phone one-handed**, or holding a child, a bag, or a cup
- **are moving around** — in a kitchen, at a counter, or away from a desk
- **order the same thing often** and want it done in one sentence instead of ten clicks

The same pattern works anywhere a form exists: appointments, support tickets, delivery
details, surveys. This page is a small, working demonstration of that idea.

---

## What it understands

| You can say | What happens |
|---|---|
| "my name is Alex" | fills the name |
| "alex at example dot com" | fills the email |
| "one coffee", "two teas", "three hot chocolates" | picks the drink and quantity |
| "paying by card", "cash" | picks the payment method |
| "place the order", "I'm done" | submits the order |
| "start over", "reset" | clears the form |

It also understands number words ("two") and plurals ("two coffees").
Anything outside this small menu is ignored on purpose — this is a demo, not a general
assistant.

---

## Optional: better accuracy with a free Gemini key

The browser's built-in speech recognition is good, but it struggles with strong accents.
If you have a free Google AI Studio key, the page can send the recorded audio to Google's
Gemini speech model instead.

1. Get a key at <https://aistudio.google.com/apikey> (free tier, no card).
2. Open the **gear icon** in the voice bar, paste the key, press **Use**.
3. Hold the mic as usual. The status changes from *Listening…* to *Transcribing…*.

The key is stored only in your browser and is sent only to Google. Press **Off** to go
back to the browser's own recognition.

This is a demo setup: because the page talks to Google directly, the key can be seen in
the browser's developer tools. Do not use a paid or shared key here.

---

## Files

| File | What it is |
|---|---|
| `index.html` | the whole app — page, styles, and logic in one file |
| `check.html` | a small microphone test page, useful if voice is not working |
| `README.md` | this file |

No build step, no server code, no database.

---

## Known limits

- Voice input needs **Chrome or Edge**, and an internet connection.
- Safari and Firefox can still use the form by hand, but the mic is disabled.
- The menu is fixed: coffee, tea, hot chocolate, quantity 1–3, card or cash.
- It is a prototype, not a product — there is no real payment and no saved orders.
- The order number is random and only shown on screen.
