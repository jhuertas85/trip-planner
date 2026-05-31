# Trip Planner

A static GitHub Pages site for family trip itineraries — no build step, no framework, just pure HTML served directly.

**Site index:** https://jhuertas85.github.io/trip-planner/

---

## Trips

| Trip | Dates | Travelers | Live page |
|------|-------|-----------|-----------|
| Namibia 2026 | May 16–31, 2026 | Juan Carlos, Maria, Ada, Felix | [Open](https://jhuertas85.github.io/trip-planner/2026-05-namibia/) |
| Germany · Switzerland · Madrid 2026 | July 2026 | Juan Carlos | [Open](https://jhuertas85.github.io/trip-planner/2026-07-switzerland/) |

---

## Features

- **Day tabs** — scrollable tab strip with at-a-glance status indicators
- **Collapsible sections** — Flight, Hotel, Activities, Meals, Warnings, Gems per day
- **Live weather** — sunrise/sunset + temperature curve (8am/1pm/8pm) via Open-Meteo (no API key needed)
- **Estimated budget** — overview card + daily cost bar broken down by hotel, activities, food, and fuel
- **Restaurant selector** — tap to pick restaurants per meal; selection is saved in sessionStorage
- **Download offline** — captures the full page as a self-contained HTML file for use without internet
- **Dark / light mode** — toggle in the header, saved to localStorage

---

## Adding a New Trip

1. Copy `template.html` into a new folder, e.g. `2027-03-japan/index.html`
2. Fill in `TRIP_DATA`, `DAY_WEATHER`, and `COST_DATA` near the top of the file
3. Update `CLAUDE.md` with the new trip summary
4. Update this README — add a row to the Trips table with the live page URL
5. Commit and push — the page is live immediately via GitHub Pages

> **Never copy an existing trip file** — always start from `template.html` to get the latest features.

---

## Structure

```
trip-planner/
├── template.html              # Master template — copy for every new trip
├── index.html                 # Trip list landing page
├── 2026-05-namibia/
│   └── index.html
├── 2026-07-switzerland/
│   └── index.html
└── CLAUDE.md                  # Codebase context and conventions
```
