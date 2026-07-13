# The Rail — Closet Cataloging

A free, single-file web app that lets a styling client photograph a few items of their closet at a time and get back clean, individually cropped, background-removed images — organized for a stylist to browse.

**Live app:** `https://YOUR-GITHUB-USERNAME.github.io/YOUR-REPO-NAME`
*(fill this in once GitHub Pages is turned on — see Setup below)*

---

## What it does

- **Client side:** upload a photo (1–3 items works best), the app finds each item, removes its background, and guesses its category — the client just glances to confirm before it's saved.
- **Stylist side:** browse every client's catalogued closet by name, filter by category, search by name/color.
- **No login/password** — just a name, so there's nothing to configure or debug on the authentication side. (Note: this also means it isn't private — anyone who knows or guesses a client's name could view their closet. Fine for early testing with trusted clients; revisit before wider launch.)

## How it works (the technical pipeline)

1. **HEIC support** — iPhones save photos as HEIC by default; the app automatically converts these to JPEG in the browser (via `heic2any`) before processing.
2. **Item detection** — real pixel-based background-subtraction segmentation finds the actual boundaries of each item in a photo (not a fixed grid guess).
3. **Manual fallback** — if a photo's background is too patterned/low-contrast for automatic detection, the app lets the person draw a box around each item by hand instead of failing outright.
4. **Background removal** — a real neural segmentation model (`@imgly/background-removal`) runs in-browser to cleanly cut out each item, with a simpler pixel-based method as a fallback if the neural model can't load.
5. **Category guessing** — a general-purpose image classifier (`MobileNet` via TensorFlow.js) suggests each item's category (tops, shoes, bags, etc.). This is a free, general-purpose model — not one trained specifically on fashion catalogs — so it's a starting guess for the human to confirm, not a final answer.
6. **Storage** — all items save to a free Supabase (Postgres) database, so the client's closet is visible to the stylist on any device.

Everything above runs for **$0** — no API keys, no per-image charges. The trade-off is accuracy: this stack is meaningfully better than nothing, but won't match paid, fashion-specific AI tools. If category accuracy or item-splitting becomes the main blocker, that's the point where a small paid API would start to pay for itself.

## Setup (one-time, ~10 minutes)

### 1. Host the app for free
- Upload `index.html` to a GitHub repository (must be named exactly `index.html`)
- Repo Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save
- Your live URL appears at the top of that page within a minute or two

### 2. Connect a free database (Supabase)
- Create a free account at [supabase.com](https://supabase.com) → New project
- In your project: **SQL Editor** → paste the SQL shown in the app's "Connect database" popup → Run
- **Project Settings → API** → copy the **Project URL** and **anon/public key**
- In the app: click "Connect database" → paste both values → Save & connect

That's it — the app is now live and saving data across devices.

## Known limitations (by design, not bugs)

- **Category accuracy varies.** Shoes, bags, and sunglasses are recognized fairly well. Jewelry is recognized poorly — the underlying model was never trained on jewelry as a concept. Always expect the human review step to matter.
- **Item splitting needs a plain background.** A patterned rug/comforter, overlapping items, or low item-vs-background contrast will trip up automatic detection — the manual box-drawing fallback exists for exactly this case.
- **No real authentication.** Names aren't passwords. Don't use this for sensitive data until real auth is added.
- **First photo of a session is slower.** The neural background-removal model (tens of MB) downloads once per session and is cached after that.

## Making changes later

This is a single self-contained HTML file — no build step, no dependencies to install locally. To edit:
1. Download `index.html` from your repo (or ask an AI coding assistant to edit it directly)
2. Make changes
3. Re-upload / commit back to GitHub — the live site updates automatically within a minute or two
