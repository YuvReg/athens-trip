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
photos, open/closed status, price level**, and a **Directions** button that
opens the native Google/Apple Maps app for walking navigation. The map also
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
| Home-base coordinates | `37.9838, 23.7220` (approx — verify exact during build) |

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
5. **Directions buttons** open the **native Google/Apple Maps app** via deep
   links — our site is for browsing, the native app is for turn-by-turn nav.
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
| **Content** | Curated lists of attractions, bars, vegan-friendly restaurants with short write-ups | ✅ Done — 42 places (17 attractions, 13 bars, 12 restaurants) |
| **The map + pins** | Google Map with a colored pin per place, grouped by category | ✅ Built (classic `google.maps.Marker`) |
| **Pin info cards** | Tap a pin → card with Google rating, reviews, photos, hours, directions | ✅ Built |
| **Live location** | Map shows the user's live position and can follow them | ✅ Built (needs https host to work) |

> **Current overall status:** the site is **built** and lives in `index.html`.
> The only thing left is the user's one manual step — create the Google API key,
> paste it in, and host it. Full guide is in `SETUP.md`. See §10 for the checklist.

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
  vegan: true,                                 // optional; marks fully-vegan spots (🌱)
  blurb: "Our own one-line write-up / why it's worth it"
}
```

### Map behavior (as built)
- Centered on the home base (§2) at a walking-distance zoom.
- Pins **color-coded by category** (attractions / restaurants / bars).
- A **🏠 orange home-base marker** for the apartment; distances/walk-times are
  measured from there.
- Tapping a pin (or a list item) opens the inline info card with live Google
  rating, reviews, photos, hours + Walking-directions / Call / Website buttons.
- A **◎ "locate me"** control + live blue dot via geolocation `watchPosition`.
- **Filters:** Attractions / Eat / Bars, plus 🌱 fully vegan, 🌅 great view,
  🚶 walkable.

---

## 6. Google API rules — cost safety & legal (READ THIS)

The user is (reasonably) worried about a surprise bill. These rules keep it at
**$0** and keep us inside Google's terms. Non-negotiable.

### Cost safety
- **Restrict the API key** to this site's domain (HTTP referrer restriction).
  A leaked key then can't be used by anyone else.
- **Restrict the key to only the APIs we use** (Maps JavaScript, Places).
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

- **Free + ~2 minutes:** Netlify Drop, GitHub Pages, or Vercel. Drag the file
  in → get an `https://` link → open it on all 4 phones.
- Everything *except* the live dot (map, pins, info cards, directions) also works
  by opening the file directly, so hosting is only strictly required for the dot.
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
- Capture each place's **exact lat/lng and Google Place ID** — the Place ID is
  what powers the live rating/photos card.
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

- [x] Curated places researched — 42 total (17 attractions, 13 bars, 12 restaurants)
- [x] Place lookup wired (live via `Place.searchByText` + `localStorage` cache — no stored IDs)
- [x] `index.html` map with Google Maps + color-coded pins + 🏠 home base
- [x] Inline info cards (live rating, reviews, photos, hours)
- [x] Live geolocation blue dot (`watchPosition`)
- [x] Directions / Call / Website deep links
- [x] Filters (Attractions / Eat / Bars + 🌱 vegan / 🌅 view / 🚶 walkable)
- [ ] **API key created, restricted, and quota-capped by the user** ← only step left (see `SETUP.md`)
- [ ] Hosted on HTTPS (Netlify/GitHub Pages/Vercel) + tested on phones
