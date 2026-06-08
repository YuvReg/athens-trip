# CLAUDE.md — Athens Trip Map Website

This file is the single source of truth for the project. Anyone (human or AI)
working in this folder should read it first. It explains **what we're building,
the decisions already locked in, how the code is organized, and the rules to
follow.** Keep it updated as the project grows.

---

## 1. What this project is (in one paragraph)

A small, good-looking **single web page** for a group trip to **Athens, Greece**.
It shows a **real Google Map** with curated pins for **attractions, bars, and
vegan-friendly restaurants** near the group's apartment. Tap a pin → a card
appears with the **real Google rating, number of reviews, review snippets,
photos, open/closed status, price level**, plus a **walking route drawn right on
our own map** (with a hand-off to Google Maps for live turn-by-turn). The map also
shows the user's **live location ("blue dot")** as they walk, like Google Maps.

It's a private trip page — 4 people, roughly a one-week stay. Not a commercial
product, not public-facing.

---

## 2. The trip facts (the "givens")

| Thing | Value |
|---|---|
| City | Athens, Greece |
| Apartment / home base | **Pl. Eleftherias 22** (Eleftherias / Koumoundourou Square) |
| Neighborhood context | Central Athens — edge of **Psyrri, Monastiraki, Metaxourgeio, Gazi** |
| Group size | 4 people |
| Trip length | ~1 week |
| Home-base coordinates | `37.98186, 23.72361` (Koumoundourou Sq center — this is the value `BASE` uses in `index.html`; fine-tune to the exact doorway if ever needed) |

**Why the location matters:** the apartment sits in the middle of Athens'
nightlife + vegan-food hotspots. Everything good should be within a **5–15 min
walk**, so the map should open zoomed to roughly that radius around the home base.

---

## 3. Locked-in decisions (do not re-litigate these)

These were already decided with the user. Build to them.

1. **Build a custom website** (not Google My Maps, not a no-code tool).
2. **Reuse Google's own map** — Google **Maps JavaScript API + Places API**.
   We are NOT building a map from scratch and NOT using Leaflet/OpenStreetMap.
3. **Vegan filter = "veg-friendly with solid vegan options."** Restaurants don't
   have to be 100% vegan, but each must have genuinely good, clearly-labeled
   vegan dishes — not just "a salad." Verify this per place during research.
4. **Inline info cards** — tapping a pin shows rating/reviews/photos/hours
   **on our page** (use Google's ready-made **Place Details** component where
   possible, so we don't hand-build the rating/review layout).
5. **Walking routes draw on our own map.** Tapping "Show walking route here"
   opens a full-screen route view (via Google's **Directions API**) with a
   **🏠 from-apartment / 📍 from-me** origin switch; a **"turn-by-turn in Google
   Maps"** deep link is the live-navigation hand-off.
   *(Supersedes the original "site is browse-only, all navigation hands off to
   the native app" decision — we now render the route ourselves and keep the
   native app only as the turn-by-turn fallback.)*
6. **Live "blue dot"** using the browser's **Geolocation** feature.
7. **Cost target = $0.** Stay inside Google's free tier; lock it down (see §6).

### The one historical note
An earlier idea was Leaflet + OpenStreetMap (free, no API key). That was
**superseded** by the decision to use Google Maps + Places API so we get real
Google ratings/reviews inline. If you see Leaflet referenced anywhere, it's
stale — Google is the chosen path.

---

## 4. The four pieces of the build

| Piece | What it means | Status |
|---|---|---|
| **Content** | Curated attractions, bars, vegan-friendly restaurants, beach/Riviera spots, football bars & practical pins | ✅ Done — 63 places (21 attractions incl. 4 beach, 18 bars incl. 3 football, 14 restaurants, 10 essentials). Count dropped 65→63 after a closure/rebrand sweep (removed Crudo + Beer Academy; renamed/repointed Mariloulou, Mama Tierra, Taf Coffee). |
| **The map + pins** | Google Map with a colored pin per place, grouped by category | ✅ Built (classic `google.maps.Marker`) |
| **Pin info cards** | Tap a pin → card with Google rating, reviews, photos, hours, directions | ✅ Built |
| **Live location** | Map shows the user's live position and can follow them | ✅ Built (works on the hosted https link) |

> **Current overall status:** the site is **built, live, and verified** (desktop +
> iPhone, zero console errors) at **https://yuvreg.github.io/athens-trip/** —
> **GitHub Pages is now the primary host** (free forever; publishing = `git push`).
> The old Netlify link https://gazi-haftaat-hatiul.netlify.app still works as a
> backup but is **no longer deployed to** (Netlify's new per-deploy "credits"
> were burning the free allowance — see §7). The Google API key is created,
> restricted (referrer allows **both** `*.netlify.app/*` and `yuvreg.github.io/*`),
> and quota-capped. The live checklist is §10; the user-facing setup guide is `SETUP.md`.

---

## 5. Tech stack & how the code is organized

**Stack:** plain **HTML + CSS + JavaScript**, single self-contained page, plus
the **Google Maps JavaScript API** and **Places API (New)**. No build tools, no
framework, no npm. The whole site is `index.html` and is editable by a non-coder.

### Actual file layout (as built)
```
trip to athens/
├─ CLAUDE.md            ← this file (project rules & decisions)
├─ index.html           ← THE WHOLE SITE — content, styling, map, logic, places list
├─ SETUP.md             ← step-by-step guide for the user (key + hosting + lockdown)
└─ .claude/             ← settings + project memory
```
Everything lives in one `index.html`. The curated places sit in a `PLACES`
array near the top (search the file for `const PLACES`) — that's the part the
user edits to add/remove/fix a spot.

**Local-only helper folders** (git-ignored, safe to delete — they're not part of
the site): `_site/` (legacy Netlify publish dir — **no longer used**; GitHub Pages
serves `index.html` straight from the repo root), `_pwtest/` (the Playwright test
script + screenshots), `.netlify/` (legacy Netlify CLI state).

### The "places" data shape (as actually built — keep this consistent)
Each place is one object in the `PLACES` array. We store only OUR own notes + a
**search `query`** — **not** Google's review/rating text (see §6 legal note),
and **not** Place IDs or coordinates. The Place ID, coordinates, rating, photos
etc. are **resolved live at runtime** by `Place.searchByText(query)` and then
**cached in `localStorage`** so we don't re-fetch on every visit.

```js
{
  cat: "attraction" | "restaurant" | "bar",   // category (note: "restaurant", not "eat")
  name: "Place name",
  query: "What to search on Google",           // drives live ID/coords/rating/photos
  area: "Neighborhood",                        // e.g. "Psyrri", "Monastiraki"
  tags: ["must", "view"],                      // optional flags used by filters
  vegan: 'full' | 'options',                   // optional; 'full' = 100% vegan (🌱),
                                               //   'options' = veg-friendly w/ solid vegan dishes
  book: "https://reservation-url",             // optional (restaurants/bars); a verified online-booking
                                               //   link (TheFork / e-restaurants / venue site). OMIT if
                                               //   walk-in/phone-only → card shows greyed "No online booking"
  wa: "+30 698 156 3511",                      // optional (restaurants/bars); the venue's VERIFIED WhatsApp
                                               //   number — a Greek +30 69… mobile. Shows a 💬 WhatsApp button
                                               //   that works on WiFi. OMIT → greyed "No WhatsApp". Landlines
                                               //   (+30 21…) are NOT on WhatsApp — never put one here.
  blurb: "Our own one-line write-up / why it's worth it"
}
```

### Map behavior (as built)
- Centered on the home base (§2) at a walking-distance zoom.
- Pins **color-coded by category** (attractions / restaurants / bars / 🛒 essentials).
- A **🏠 house-shaped home marker** for the apartment (an inline-SVG icon, not a
  plain dot); tap it to draw a walking route home from your live location.
  Distances/walk-times are measured from there.
- Tapping a pin (or a list item) opens the inline info card with live Google
  rating, reviews, photos, hours + Walking-route / 💬 WhatsApp / Website buttons.
  (The old `tel:` **Call** button was removed — the group is on WiFi only and can't
  make cellular calls; WhatsApp messaging/calls work over WiFi. Restaurant/bar cards
  show a green WhatsApp button where the venue has a verified `wa` number, else a
  greyed "No WhatsApp". See the `wa` field above + §10.)
- Three stacked map controls (bottom-right): **◎ locate me** (live blue dot via
  geolocation `watchPosition`), **🏠 walk-me-home** (routes to the apartment from
  your live location — origin is always "you", so the from/to switch is hidden),
  and **📏 ruler** (tap 2+ points to measure walking distance; reuses `haversine`).
- **Filters (smart / context-sensitive):** category chips Attractions / Eat /
  Bars / 🛒 Essentials; always-on toggles ⭐ favorites, 🟢 open now, 🚶 walkable;
  plus context toggles that appear only for their category — 🌱 fully vegan &
  🥗 vegan options (under Eat), ⚽ football (under Bars), 🏖️ beach, 🌅 great view.
  "Open now" is computed in Athens time from cached opening hours (lazy-fetched).
  Far spots (beaches / ride-outs) show tram/taxi distance + hand routing to the
  Google Maps app instead of drawing a nonsensical 20 km walk.
- **Extras:** a ☆ **favorite** star on every place + a **"Share my picks"**
  button (sends the starred shortlist to the group chat); an **offline fallback
  card** when Google can't load; a **🌅 golden-hour** badge in the late-afternoon
  window; and the in-site full-screen **route mode** described in §3.

---

## 6. Google API rules — cost safety & legal (READ THIS)

The user is (reasonably) worried about a surprise bill. These rules keep it at
**$0** and keep us inside Google's terms. Non-negotiable.

### Cost safety
- **Restrict the API key** to this site's domain (HTTP referrer restriction).
  A leaked key then can't be used by anyone else.
- **Restrict the key to only the APIs we use** (Maps JavaScript, Places **(New)**,
  and **Directions API** — Directions powers the in-site walking-route preview; it
  has its own generous free tier and stays $0 at our volume).
- **Set a hard daily quota cap** in Google Cloud Console (e.g. ~1,000
  requests/day). If somehow hit, the API just stops responding — **it cannot
  generate a charge.** Worst case = "map stops loading for a day," never a bill.
- At our scale (4 people, a few dozen places, one week) usage is a rounding
  error against the free tier.

### Legal / terms
- **Do NOT permanently store Google's ratings, reviews, or photos** in our code
  or files. Google's terms require this content to be **fetched live**. We may
  store our own notes and the **Place ID** only. Fetching live is free at our
  volume, so this costs nothing — just build it the clean way.

### The API key itself
- The key is **the user's to create** (needs their Google login + a card on
  file to enable billing — which stays at $0 via the caps above).
- **Never hardcode a real key into a committed file** without the user's
  go-ahead, and never share it publicly. Use a clearly-marked placeholder like
  `YOUR_GOOGLE_MAPS_API_KEY` until the user pastes theirs in.

---

## 7. Hosting (needed for the live blue dot)

The live location dot **only works over HTTPS** (a browser privacy rule). So the
finished page must be **hosted**, not just emailed around.

### Current host: GitHub Pages (the live link the group uses)
- **Live link:** **https://yuvreg.github.io/athens-trip/** — served by **GitHub
  Pages** from the repo `YuvReg/athens-trip` (branch `main`, root folder). The repo
  is **public** (safe: the only key in it is the referrer-restricted, public-by-design
  Maps key — already visible on any live Maps site).
- **How we publish now (THE deploy path — do this, nothing else):** edit
  `index.html` → `git add -A && git commit && git push`. GitHub Pages **auto-rebuilds
  in ~1 min**. That's it. **Cost = $0, unlimited, forever.** No CLI, no `_site/` copy,
  no per-deploy cost.
- **⚠️ Do NOT deploy to Netlify anymore.** Netlify switched to a per-deploy "credits"
  model and ~14 test deploys burned almost the whole monthly free allowance
  (210 credits). It never charged money (free plan, no card → it just pauses), but
  it's a needless limit. The old Netlify link
  (https://gazi-haftaat-hatiul.netlify.app) is kept **only as a read-only backup** —
  leave it serving, don't push new deploys to it.

- **Caveat (important):** once the API key is referrer-restricted to the hosted
  domain (which it is), **opening `index.html` directly from disk (`file://`) will
  make Google reject the map with a 403** — a `file://` page sends no HTTP referrer
  for the key to match. So in practice everyone must use the **hosted GitHub Pages
  link**, not the raw file. (Before the key is restricted, the file opens fine locally.)
- **Key referrer list must include both domains:** `yuvreg.github.io/*` (primary)
  and `*.netlify.app/*` (backup). Adding the github.io entry was the one-time manual
  step done on 2026-06-08. If the map ever shows blank on a new host, it's almost
  always a missing referrer entry.
- For offline use, don't try to build offline maps (fiddly). Instead rely on the
  "Open in Google Maps" buttons + tell the group to pre-download Athens in the
  Google Maps app.

---

## 8. Research standard (the real effort of this project)

The hard part isn't the code — it's curating genuinely good, currently-open
spots. When researching places:

- **Verify each place is real and currently open** (hours change, places close).
- **Restaurants:** confirm solid, clearly-labeled vegan options exist (per §3).
  HappyCow and recent reviews are good sources.
- **Bars:** real, well-reviewed local spots — not tourist traps.
- **Attractions:** the genuinely worth-it sights, prioritized by closeness to
  the home base.
- For each place, nail down a **search `query`** specific enough that
  `Place.searchByText` resolves the *correct* spot (right branch, right city).
  We deliberately **don't** store lat/lng or Place IDs — they're resolved live
  from the query at runtime (see §5) — so getting the query right is the job.
- Prefer places within a **5–15 min walk** of Pl. Eleftherias 22; note when
  something further is worth the trip.

Allowed research domains are pre-approved in `.claude/settings.local.json`
(currently `happycow.net`, `realgreekexperiences.com`, plus WebSearch).

---

## 9. Working rules for anyone editing this project

- **Keep all files inside this project folder.** No scratch files in temp dirs.
- **The places list is the user-facing knob** — keep it tidy, commented, and
  easy for a non-coder to edit (one place per object, plain-English notes).
- **Explain choices in plain language** in code comments where a non-coder might
  look — the user is newer to the technical world and wants to understand it.
- **Don't introduce frameworks/build tools** without a reason — the whole point
  is a simple, single, hand-editable page.
- **Update this CLAUDE.md** when a real decision changes (and mark the old one
  as superseded, like we did with Leaflet in §3).

---

## 10. Quick status snapshot

> Update this section as work progresses so anyone opening the project knows
> where things stand.

- [x] Curated places — 63 total (21 attractions incl. 4 beach, 18 bars incl. 3 football, 14 restaurants, 10 essentials)
- [x] **Data accuracy sweep (closures/rebrands)** via `_pwtest/audit.mjs` (compares each place's live Google name + `businessStatus` to ours): renamed Los Vegans→**Mariloulou**, repointed **Mama Tierra**→Acropolis branch, **Dope Roasting**→**Taf Coffee**, removed **Crudo** (now a fish taverna) + **Beer Academy** (→"beertime", unverified for football); kept **six d.o.g.s** (Google "closed" flag is stale). Re-run `audit.mjs` before a trip to re-check. Temp-closed to watch: Kerameikos, Avocado, A for Athens, Holy Spirit.
- [x] Place lookup wired (live via `Place.searchByText` + `localStorage` cache — no stored IDs)
- [x] `index.html` map with Google Maps + color-coded pins + 🏠 home base
- [x] Inline info cards (live rating, reviews, photos, hours)
- [x] Live geolocation blue dot (`watchPosition`)
- [x] Directions / Call / Website deep links
- [x] **In-site walking route** — tap "Show walking route here" → full-screen map +
      turn-by-turn, with a **🏠 from apartment / 📍 from my location** origin switch
      and a ← Back button (Directions API; falls back to Google Maps if it ever fails)
- [x] **🍽️ "Book a table" button** on restaurant/bar cards — active reservation link when the place has
      verified online booking (`book` field; 6 set: Line/Noel/Bolivar on i-host, MoMix/360/Krabo on own
      sites), else greyed "No online booking". Header personalised to "Athens ~ Gazi Al Tadlik Et Duma".
- [x] **💬 WhatsApp contact button** — replaces the old `tel:` Call on restaurant/bar cards (the group is
      WiFi-only and can't make cellular calls; WhatsApp works over WiFi). Green `wa.me` deep link where the
      venue has a VERIFIED WhatsApp mobile (`wa` field; 5 set: The Place, Kokkion, The Bar in Front of the
      Bar, The Lucky Sparrow, Bolivar), else greyed "No WhatsApp". Most Athens spots are landline-only (NOT
      on WhatsApp) — each `wa` verified per-venue from its own IG/site; landlines (+30 21…) never qualify.
      Verified live desktop + iPhone (0 console errors), 2026-06-08.
- [x] **🏠 House home-marker + walk-me-home + 📏 ruler** — apartment is a house-shaped
      inline-SVG marker; tap it OR the new 🏠 button (stacked above ◎ locate, bottom-right)
      to route home from your live location; 📏 ruler button measures distance between
      tapped points (`haversine`). Verified live on desktop + iPhone (0 console errors), 2026-06-08.
- [x] **Smart filters** — chips Attractions / Eat / Bars / 🛒 Essentials + context toggles
      (⭐ favorites, 🟢 open now, 🌱 fully vegan, 🥗 vegan options, ⚽ football, 🏖️ beach, 🌅 view, 🚶 walkable)
- [x] **Beach / Riviera** spots (Glyfada / Vouliagmeni / Alimos) — far places show tram/taxi + Google-Maps transit hand-off
- [x] **Football bars** for the World Cup (Athens Sports Bar, James Joyce, Lucky Sparrow — web-verified; Beer Academy dropped in the accuracy sweep)
- [x] **Open now** badges + filter (Athens-time, computed from cached opening hours, lazy-fetched)
- [x] **Practical pins** (supermarket, central market, pharmacy, metros, late-night food, coffee, ATM)
- [x] **API key created, restricted (referrer + APIs), quota-capped** by the user
- [x] **Hosted on HTTPS** → **https://yuvreg.github.io/athens-trip/** (GitHub Pages — primary,
      free-forever, publish = `git push`). Old https://gazi-haftaat-hatiul.netlify.app kept as backup.
      Verified working on the github.io link (desktop + iPhone, 0 console errors) on 2026-06-08.
- [x] One-finger map gestures (`greedy`); verified live on desktop + iPhone via Playwright
- [ ] Final on-the-phone check by the group during the trip (nice-to-have)
