# Trip Planner — Claude Context

## What this repo is
A static GitHub Pages site for trip itineraries. No build step, no framework — pure HTML files served directly.

**GitHub repo:** https://github.com/jhuertas85/trip-planner
**Live site:** https://jhuertas85.github.io/trip-planner/

## File structure
```
trip-planner/
├── template.html              # Master template — copy this for every new trip
├── 2026-05-namibia/
│   └── index.html             # Namibia May 2026 (Juan Carlos, Maria, Ada, Felix)
└── CLAUDE.md                  # This file
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
  activities: [{ time, text, subNote, needsBooking }],
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
- **New trip:** copy `template.html` into a new folder e.g. `2027-03-japan/index.html`, then fill in `TRIP_DATA`, `DAY_WEATHER`, and `COST_DATA`
- **After any edit:** commit + push using the token pattern above

## Namibia 2026 trip summary
- **Travellers:** Juan Carlos, Maria, Ada, Felix
- **Dates:** May 16–31, 2026 (16 days, Dubai ↔ Addis ↔ Namibia)
- **Flights:** Ethiopian Airlines (ET 601/835/834/602)
- **Car rental:** Namibia2Go · 2,000 EUR confirmed
- **Ethiopia tour:** Talelign Befkadu · USD 480 total (Day 1: Addis city, Day 2: Bishoftu crater lakes)
- **Route:** Addis Ababa (2 nights) → Sossusvlei → Swakopmund (4 nights) → Damaraland → Grootberg → Etosha (2 nights) → Windhoek → home
