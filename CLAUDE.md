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

Each trip is a self-contained single HTML file. All trip data lives in a `TRIP_DATA` JavaScript object near the top of the rendering script. The rendering engine reads from that object — **only edit `TRIP_DATA`, `DAY_WEATHER`, and `COST_DATA` for content changes**, not the rendering code.

## Template-first rule
**Any new feature or structural improvement added to one trip must also be applied to `template.html` immediately.** When creating a new trip, always copy `template.html` — never copy an existing trip file.

## How to push changes
Git push via the local proxy returns 403. Use a GitHub Personal Access Token (classic, `repo` scope):

```bash
git add <file> && git commit -m "description"
git push https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main
git fetch https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main:refs/remotes/origin/main
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
  dayNumber, date, dayOfWeek, title, location, backgroundImage,
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

## Features in every trip (all come from template)
- **Day tabs** — scrollable horizontal tab strip with missing-status indicator
- **Collapsible sections** — Flight, Hotel, Activities, Meals, Warnings, Gems, Weather, per day
- **Live weather** — sunrise/sunset + temperature curve (8am/1pm/8pm) via Open-Meteo
- **Estimated budget** — overview card + daily cost bar (🏨 hotel · 🗺️ activities · 🍽️ food · ⛽ fuel)
- **Restaurant selector** — tap to select restaurants per meal, persisted in sessionStorage
- **Download offline** — ⬇ button captures full page as a self-contained HTML file
- **Dark/light mode** — toggle in header, saved to localStorage

## Common tasks
- **Update a hotel:** find the day by `dayNumber` in `TRIP_DATA.days`, edit the `hotel` object
- **Add/change activity:** find the day, edit the `activities` array
- **New trip:** copy `template.html` into a new folder e.g. `2027-03-japan/index.html`, then fill in `TRIP_DATA`, `DAY_WEATHER`, and `COST_DATA`, **and update the File structure section in this CLAUDE.md** with the new folder name, dates, and travelers
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
