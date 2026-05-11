# Trip Planner — Claude Context

## What this repo is
A static GitHub Pages site for trip itineraries. No build step, no framework — pure HTML files served directly.

**Live site:** https://jhuertas85.github.io/trip-planner/

## File structure
```
trip-planner/
├── template.html              # Reusable blank template for new trips
├── 2026-05-namibia/
│   └── index.html             # Namibia May 2026 group trip (16 travellers: Juan Carlos, Maria, Ada, Felix)
└── CLAUDE.md                  # This file
```

Each trip is a self-contained single HTML file. All trip data lives in a `TRIP_DATA` JavaScript object near the top of the file. The rendering engine at the bottom reads from that object — **only edit `TRIP_DATA` for content changes**, not the rendering code.

## How to push changes
Git push via the local proxy returns 403. Use a GitHub Personal Access Token (classic, `repo` scope):

```bash
git add <file> && git commit -m "description"
git push https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main
git fetch https://jhuertas85:TOKEN@github.com/jhuertas85/trip-planner.git main:refs/remotes/origin/main
```

The user generates a token at GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic), pastes it in chat, and it gets used once then revoked.

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

## Namibia 2026 trip summary
- **Travellers:** Juan Carlos, Maria, Ada, Felix
- **Dates:** May 16–31, 2026 (16 days, Dubai ↔ Addis ↔ Namibia)
- **Flights:** Ethiopian Airlines (ET 601/835/834/602)
- **Car rental:** Namibia2Go (Gondwana Collection)
- **Route:** Addis Ababa (2 nights) → Sossusvlei → Swakopmund (4 nights) → Damaraland → Grootberg → Etosha (2 nights) → Windhoek → home

## Common tasks
- **Update a hotel:** find the day by `dayNumber` in `TRIP_DATA.days`, edit the `hotel` object
- **Add/change activity:** find the day, edit the `activities` array
- **New trip:** copy `template.html` into a new folder e.g. `2027-03-japan/index.html`, update `TRIP_DATA`
- **After any edit:** commit + push using the token pattern above
