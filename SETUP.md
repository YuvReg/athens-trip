# Athens Trip Map — Setup Guide 🇬🇷

Your website is the file **`index.html`** in this folder. It already has all your places
(attractions, bars, vegan-friendly restaurants) built in. It just needs **one thing from
you** to switch on the map and the live ratings: a free **Google key**.

This guide has 4 short parts. Total time: **about 15 minutes**, once.

> **What's a "key"?** Think of it as a password that lets *your* page ask Google for its
> map, ratings, reviews and photos. Google requires a card on file to hand one out, but
> your usage (4 people for a trip) sits comfortably inside Google's free allowance — you
> will realistically pay **€0**. Part 4 shows how to lock it down so it *stays* free.

---

> **What does "card on file" mean? (Important — read this!)**
> "On file" just means *saved to your account*. You are giving Google your card details so
> they are **stored**, ready in case a charge ever happens — but **saving the card is NOT a
> payment.** Nothing is charged just for adding it. It's exactly like a hotel taking your
> card at check-in: they keep it on record for *just-in-case* extras, but if you don't use
> any, you leave having paid nothing. Google only ever charges if you go *past* the big free
> allowance — which 4 people on a trip never will. So yes, you understood right: you add your
> credit card to your new Google account, and it simply sits there unused.
>
> 👉 One thing not to panic about: right after adding the card you may see a **€0–€1
> "pending" line** on your bank app. That's just your bank confirming the card is real — it
> disappears on its own. It is **not** a charge for the service.

## Part 1 — Get your free Google key (~10 min)

**Have your credit or debit card nearby before you start.**

1. **Sign in.** Go to **https://console.cloud.google.com** and sign in with your Google
   account. If it's your first time, pick your country, tick the **Terms of Service** box, and
   click **Agree and Continue**.

2. **Create a project** (a "project" is just a labelled box that holds your key).
   - At the very top of the page, click the project dropdown (it says *"Select a project"*).
   - Click **New Project** (top-right of the little window).
   - Name it `Athens Trip`, leave everything else as-is, click **Create**.
   - After a few seconds, open the dropdown again and **click your new project** to select it.

3. **Turn on billing** — *this is the "card on file" step* (see the box above; it's safe).
   - Click the **☰ menu** (three lines, top-left) → **Billing**.
   - Click **Link a billing account** → **Create billing account** (or **Manage billing
     accounts → Create account**).
   - Choose your **country** and **currency**, click **Continue**.
   - For account type choose **Individual**.
   - Type your **name + address**, then your **card number, expiry and CVC**.
   - Click **Start free** / **Submit and enable billing**.
   - ✅ Your card is now saved ("on file"). Remember: no real charge happens here.

4. **Switch on the two map services.** Click **☰ menu → APIs & Services → Library**. Then,
   one at a time, search each name below, click the result, and click the blue **Enable**
   button:
   - **Maps JavaScript API**  ← draws the map
   - **Places API (New)**  ← supplies the ratings, reviews, photos and hours

5. **Create your key.** Click **☰ menu → APIs & Services → Credentials**, then
   **+ Create credentials → API key**. A box pops up with a long code like
   `AIzaSyB....`. Click the **copy icon**. 🎉 That's your key — paste it somewhere safe for a
   moment (you'll need it in Part 2).

   *(Leave the "restrict key" option for Part 4 — we'll lock it once you have your website
   address. Click **Close** for now.)*

---

## Part 2 — Put the key into your website (~1 min)

1. Open **`index.html`** in a text editor.
   - On Windows: right-click the file → **Open with → Notepad** (or, nicer, free
     [Notepad++](https://notepad-plus-plus.org/) or [VS Code](https://code.visualstudio.com/)).
2. Very near the top you'll see this line:
   ```
   window.GOOGLE_MAPS_API_KEY = "YOUR_GOOGLE_MAPS_API_KEY";
   ```
3. Replace **`YOUR_GOOGLE_MAPS_API_KEY`** with the key you copied — **keep the quotation
   marks**. It should end up looking like:
   ```
   window.GOOGLE_MAPS_API_KEY = "AIzaSyB...your real key...";
   ```
4. **Save** the file.

---

## Part 3 — Your site is already online (just share the link)

The live "blue dot" (your location) only works when the page is online with an `https://`
address — that's a privacy rule in every phone. **Good news: your site is already hosted,
free, on GitHub Pages — nothing to do here.**

- **Your website link — share this in the group chat:**
  **https://yuvreg.github.io/athens-trip/**
- Open it on all 4 phones, add it to the home screen, and you're set.
- **Updates are automatic and free forever:** when a change is saved to the project on
  GitHub, the live link refreshes itself in about a minute. No re-uploading, no usage
  limits, no cost.

> **Why GitHub Pages and not Netlify?** We started on Netlify, but Netlify now charges
> "credits" every time the site is re-published, and testing ate almost the whole monthly
> free allowance. GitHub Pages re-publishes for **free, unlimited.** The old Netlify link
> (https://gazi-haftaat-hatiul.netlify.app) still works as a backup, but the GitHub link
> above is the main one.

---

## Part 4 — Lock the key so it stays free & safe (~3 min)

This stops anyone else from using your key, and makes a surprise bill effectively impossible.

1. Back in Google Cloud: **APIs & Services → Credentials → click your key's name.**
2. Under **Application restrictions**, choose **Websites**, then click **Add** and enter your
   site address(es) with `/*` on the end. The live site uses:
   ```
   https://yuvreg.github.io/*
   ```
   *(the old backup `*.netlify.app/*` is also kept in the list — leave it).* This means the
   key only works *from your site* — a copied key is useless to anyone else. **This is already
   set up** (the github.io line was added on 2026-06-08).
3. Under **API restrictions**, choose **Restrict key** and tick only:
   **Maps JavaScript API** and **Places API (New)**. Click **Save**.
4. **Peace-of-mind cap (optional but recommended):** go to **Billing → Budgets & alerts →
   Create budget**, set it to something tiny like **€1**, and you'll get an email the instant
   anything is ever charged. For a true hard stop, you can also lower the daily request limit
   under each API's **Quotas** page — but honestly, your trip won't get near the free limit.

That's it — your map is live. 🗺️

---

## What you get

- **Tap any place** (in the list or a pin) → its real Google **rating, reviews, photos,
  hours**, plus **Walking directions**, **Call** and **Website** buttons.
- **Filters:** Attractions / Eat / Bars, plus 🌱 Fully vegan, 🌅 Great view, 🚶 Walkable.
- **The ◎ button** on the map shows your **live location** as you walk — like Google Maps.
- **🏠 orange pin** = your apartment; every distance and walk-time is measured from there.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Page says **"add your Google key"** | The key in `index.html` is still the placeholder, or was pasted without quotes. Re-do Part 2 and re-upload. |
| Map area is **grey / "can't load Google Maps"** | A service isn't enabled or billing isn't on. Recheck Part 1 steps 3–4 (both APIs **Enabled**, billing active). New keys can take a few minutes to activate. |
| Ratings/photos don't appear | Make sure **Places API (New)** is enabled, and that your key restriction (Part 4) includes it. |
| **Blue dot** doesn't show | (1) You must open the **Netlify https link**, not the file on your computer. (2) Tap **Allow** when the phone asks for location. (3) Works best outdoors. |
| Want to add or remove a place | In `index.html`, find the `PLACES` list and copy an existing line — set `name`, `query` (what to search on Google), `cat` (`attraction`/`restaurant`/`bar`) and a `blurb`. Re-upload. |

Have a fantastic trip! 🍷🏛️🌱
