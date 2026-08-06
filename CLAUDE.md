# Trip Planner — Claude Context

## What this repo is
A static GitHub Pages site for trip itineraries. No build step, no framework — pure HTML files served directly.

**GitHub repo:** https://github.com/jhuertas85/trip-planner
**Live site:** https://jhuertas85.github.io/trip-planner/

## File structure
```
trip-planner/
├── index.html                 # Homepage — trips listing page (GitHub Pages root)
├── template.html              # Master template — copy this for every new trip
├── favicon.png
├── README.md
├── CLAUDE.md                  # This file
├── tickets/                   # PDF travel tickets
│   ├── DB_Munich-Zurich_30Jun2026.pdf
│   ├── Renfe_Pamplona-Madrid_7Jul2026.pdf
│   └── SwissTravelPass_1-5Jul2026.pdf
├── 2026-05-namibia/
│   └── index.html             # Namibia May 2026 (Juan Carlos, Maria, Ada, Felix)
├── 2026-07-switzerland/
│   └── index.html             # Germany · Switzerland · Madrid July 2026 (Juan Carlos)
├── 2026-07-puerto-rico/
│   └── index.html             # Puerto Rico Jul 2026 (10 people · Luquillo · Jul 8–12)
└── 2026-07-marias-trip/
    └── index.html             # Maria's Trip July–Aug 2026 (Dubai · Madrid · Puerto Rico · Kraków · London)
```

Each trip is a self-contained single HTML file. All trip data lives in four JavaScript objects near the top of the rendering script. The rendering engine reads from those objects — **only edit `TRIP_DATA`, `DAY_WEATHER`, `COST_DATA`, and `DAY_CURRENCY` for content changes**, not the rendering code.

## Template-first rule
**Any new feature or structural improvement added to one trip must also be applied to `template.html` immediately.** When creating a new trip, always copy `template.html` — **NEVER copy an existing trip file** (e.g. the Switzerland trip has custom Leaflet maps, specific route code, and old rendering — copying it will silently break the new trip with Germany maps and wrong layouts).

## How to push changes
**Always commit and push directly to `main`.** Never leave changes on a feature branch — the live site only serves from `main`, so changes on other branches are invisible.

Git push via the local proxy returns 403. Use a GitHub Personal Access Token (classic, `repo` scope):

```bash
git add <file> && git commit -m "description"
git push https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main
git fetch https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main:refs/remotes/origin/main
```

If working on a branch (e.g. assigned by the session), merge it into main before finishing:
```bash
git checkout main
git merge --ff-only <branch-name>
git push origin main
```

## TRIP_DATA structure (top-level)
```js
const TRIP_DATA = {
  title: "Trip Name 2027",
  travelers: ["Person 1", "Person 2"],

  // Optional — set to null to hide the section entirely
  carRental: { company, phone, website } | null,
  spotifyPlaylist: "https://open.spotify.com/..." | null,
  tricountUrl: "https://tricount.com/..." | null,
  checkInUrl: "https://airline.com/check-in" | null,

  flights: [{ date, dayOfWeek, airline, from, to, departTime, arriveTime, terminal, passengers }],
  days: [ ...see per-day structure below... ]
}
```

## TRIP_DATA structure (per day)
```js
{
  dayNumber,          // integer, starts at 1 — NEVER use the itinerary's own day number
  date,               // "Aug 15" format ONLY — abbreviated month + space + day (no year, no comma)
  dayOfWeek,          // "Mon" / "Tue" / "Wed" / "Thu" / "Fri" / "Sat" / "Sun" — NEVER empty
  title,              // short label e.g. "Arrival" or "Uluwatu & Seminyak"
  location,           // city/region e.g. "Bali, Indonesia"
  backgroundImage,    // Unsplash URL: "https://images.unsplash.com/photo-XXXXX?w=800"
                      // Keep the template default if unsure — a repeated image is fine, a broken URL shows black
  flight: { airline, from, to, departTime, arriveTime, terminal } | null,
  hotel: { name, status, note, bookedBy, mapsUrl, altHotel? },
  drive: { from, to, duration, distance } | null,
  activities: [{ time, text, subNote, needsBooking, mapsUrl? }],  // mapsUrl optional; auto-fallback searches "text + location"
  meals: {
    breakfast/lunch/dinner: { status, note, restaurants: [{ name, desc, rating, mapsUrl }] }
  },
  route: "maps.google.com/..." | null,
  warnings: [...strings],
  gems: [...strings]
}
```

**Hotel status values:** `confirmed` | `pending` | `missing`
**Meal status values:** `confirmed` | `included` | `options` | `pending` | `flexible` | `missing`

## DAY_WEATHER structure
Defined separately from TRIP_DATA. Provides coordinates for the live weather widget (Open-Meteo, free, no key).
```js
const DAY_WEATHER = {
  1: { lat: 9.0247, lon: 38.7468, isoDate: "2026-05-16" },
  // one entry per day number
};
```
Leave as `{}` if weather is not needed. Data is cached 3 hours in localStorage; ↻ button forces refresh.

## COST_DATA structure
```js
const COST_DATA = {
  disclaimer: "...",
  summary: [{ cat, amount, note }],   // shown in overview budget card
  days: {
    1: { hotel, activities, food, other },  // shown as cost bar on each day
  }
};
```
Set all amounts to 0 and fill in as bookings are confirmed.

## DAY_CURRENCY structure
Defined separately. Maps day number → local currency for that day. The calculator always shows AED / USD / EUR, plus a 4th row for the local currency when it differs from those three.
```js
const DAY_CURRENCY = {
  1: { code: 'CHF', name: 'Swiss Franc', symbol: 'CHF' },
  // Same entry can repeat for consecutive days in the same country
};
```
Leave as `{}` if the trip uses only AED / USD / EUR. Use ISO 4217 currency codes (JPY, THB, GBP, etc.).

## Features in every trip (all come from template)
- **Day tabs** — scrollable horizontal tab strip with missing-status indicator; today's tab is highlighted in gold
- **Collapsible sections** — Flight, Hotel, Activities, Meals, Warnings, Gems, Weather, Currency, per day
- **Live weather** — sunrise/sunset + temperature curve (8am/1pm/8pm) + rain probability via Open-Meteo; dual cache (30 min for current days, 3h for future)
- **Currency calculator** — AED / USD / EUR always shown; 4th row added automatically for local currency (set via `DAY_CURRENCY`); live rates from open API with offline fallback
- **Estimated budget** — overview card + daily cost bar (🏨 hotel · 🗺️ activities · 🍽️ food · ⛽ fuel); amounts in AED
- **Restaurant selector** — tap to select restaurants per meal, persisted in sessionStorage
- **Download offline** — ⬇ button captures full page as a self-contained HTML file
- **Dark/light mode** — toggle in header, saved to localStorage
- **PWA support** — manifest.json + service worker for "Add to Home Screen" (copy `sw.js` + `manifest.json` from an existing trip folder into the new trip folder, then update `manifest.json` with the trip name)

## Common tasks
- **Update a hotel:** find the day by `dayNumber` in `TRIP_DATA.days`, edit the `hotel` object
- **Add/change activity:** find the day, edit the `activities` array
- **New trip:**
  1. Copy `template.html` verbatim into a new folder e.g. `2027-03-japan/index.html`
  2. **ONLY edit the first `<script>` block** — the one that starts right after `<body>` and ends with `// END OF DATA — DO NOT EDIT BELOW THIS LINE`. Everything after that marker is the rendering engine; never touch it.
  3. In that first `<script>`, fill in all four data objects: `TRIP_DATA`, `DAY_WEATHER`, `COST_DATA`, `DAY_CURRENCY`
  4. **`dayNumber` must start at 1 and increment by 1** for each day — never use the trip itinerary's own day numbering (e.g. if the itinerary says "Day 8: Munich" but it's the first day of this trip file, set `dayNumber: 1`)
  5. **`TRIP_DATA.flights` must be an array** — use `[]` if no flights yet, never `null`
  6. **`TRIP_DATA.travelers` must be an array** — use `["Traveler"]` as minimum, never `null`
  7. **Every day must have all three meal types** (`breakfast`, `lunch`, `dinner`) in `day.meals`, even if just `{ status: "flexible", note: "", restaurants: [] }` — omitting any one crashes the page
  8. Update the **File structure section** in this CLAUDE.md with the new folder name, dates, and travelers
  9. Add a trip summary section at the bottom of this CLAUDE.md
- **After writing a new trip file, validate JS syntax before pushing** — run this from the repo root (replace the path with the actual file):
  ```bash
  node -e "
  const c = require('fs').readFileSync('2027-03-japan/index.html','utf8');
  const m = c.match(/<script>([\s\S]*?)<\/script>/);
  try { new Function(m[1]); console.log('✓ Syntax OK'); }
  catch(e) { console.error('✗ Syntax error:', e.message); process.exit(1); }
  "
  ```
  Fix any error before pushing. Never push a file with a syntax error — the PWA won't show error details.
- **After any edit:** commit + push using the token pattern above

## Rule: keep this file current
Whenever a new trip folder is created or an existing one is significantly updated (travelers, dates, route), update the **File structure** section above and add or revise the matching trip summary section at the bottom of this file. This ensures every Claude session starts with accurate context without needing to scan the repo.

## Puerto Rico 2026 trip summary
- **Travellers:** Mamá (76), Papá (81), Giova, Ari (23), Coki, Liz, Emma (8), Matías (16), JC, Maria — 10 total
- **Dates:** Jul 8–12, 2026 (5 days · Mié–Dom)
- **Base:** Villa Pitahaya (Airbnb 827071, Luquillo) · 10 guests · Piscina · Confirmado · GPS 18.3756,-65.7225
- **Flights:** Iberia IB381 MAD→SJU Jul 8 (12:45–15:25, JC & Maria) · IB382 SJU→MAD Jul 12 (17:15, JC & Maria)
- **Car rental:** Alamo × 2 Midsize SUVs · Jul 8 4pm pickup → Jul 12 3pm return · (833) 763-1735
- **Route:** Playa Luquillo (Día 1) → Cuevas Camuy mañana (grupo activo) + VSJ todos 2pm (Día 2) → El Yunque día completo (Día 3) → Playas + actividades (Carabalí/Seven Seas/Cayo Icacos) + Bio Bay 9:45pm (Día 4) → Salida (Día 5)
- **Lenguaje:** 100% español · USD para todas las transacciones
- **Bookings necesarios:** Lancha Bio Bay (efectivo, teléfono) · Carabalí rides (reserva anticipada)

## Namibia 2026 trip summary
- **Travellers:** Juan Carlos, Maria, Ada, Felix
- **Dates:** May 16–31, 2026 (16 days, Dubai ↔ Addis ↔ Namibia)
- **Flights:** Ethiopian Airlines (ET 601/835/834/602)
- **Car rental:** Namibia2Go · 2,000 EUR confirmed
- **Ethiopia tour:** Talelign Befkadu · USD 480 total (Day 1: Addis city, Day 2: Bishoftu crater lakes)
- **Route:** Addis Ababa (2 nights) → Sossusvlei → Swakopmund (4 nights) → Damaraland → Grootberg → Etosha (2 nights) → Windhoek → home

## Maria's Trip 2026 summary
- **Traveller:** Maria
- **Dates:** Jul 5 – Aug 2, 2026 (29 days · Sun–Sun)
- **Route:** Dubai → Kraków → Madrid (Jul 5–8) → Puerto Rico (Jul 8–12) → Madrid (Jul 12–14) → Kraków (Jul 14–26) → Warsaw (Jul 18–19) → London (Jul 19–23) → Kraków (Jul 24–31) → Dubai (Aug 2)
- **Flights:** FZ 1787, LO 2161, IB 0379, IB 0382, LH 1805, LH 1626, LO 279, LO 280, FZ 1788 (9 flights total)
- **Puerto Rico base:** Camino Del Lago Pitahaya, Luquillo (Jul 9–12)
- **Kraków:** Dom Franciszkański San Antonio (Jul 15–18), Campanile Prime Kraków Old Town (Jul 24–31)
- **Activities:** Old San Juan, El Yunque Rainforest, Hacienda Carabalí, Bioluminescent Bay, Remote work in Madrid/London, Time with Alex in Kraków
- **Work locations:** Nomia office (Kurniki 9, Kraków), Nomia office (10 York Road, London)
- **Trains:** Kraków–Warsaw (Jul 18), Warsaw–Kraków (Jul 24)
