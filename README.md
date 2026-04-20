# Resource Atlas 

An interactive geomap for exploring, filtering, routing to, and analyzing point-based resources (temples, schools, hospitals, tourism spots, etc.) from a JSON dataset. Built with Leaflet + OSRM, no build step required.

---

## 1. Files

```
your-folder/
├── map.html         ← the app (open this in a browser via a local server)
├── locations.json   ← your data (name, latitude, longitude, type)
└── csv_json.ipynb   ← optional: converts a CSV to locations.json
```

`locations.json` must be an array of objects in this shape:

```json
[
  {
    "name": "Omkareshwar Temple Ukhimath",
    "latitude": 30.51862253,
    "longitude": 79.0954182,
    "type": "temple"
  }
]
```

`type` drives the marker icon, color, and category filter. Known types include `hospital`, `toilet`, `temple`, `school`, `market`, `police`, `hotel`, `tourism`, `office`, `village`, `transport`, `bank`, `atm`, `restaurant`, `fuel`, `pharmacy`, `post`. Any unknown type gets a generic grey pin — the app still works.

---

## 2. Running it locally

`map.html` uses `fetch()` to load `locations.json`, which **does not work** from a `file://` URL — you need a local HTTP server. From the folder containing both files:

**Python (easiest):**
```bash
python -m http.server 8000
```
Then open http://localhost:8000/map.html

**Node alternatives:**
```bash
npx serve
# or
npx http-server
```

**VS Code:** install the *Live Server* extension, right-click `map.html` → *Open with Live Server*.

---

## 3. Updating the data

If you have a CSV (`name, latitude, longitude, type`), convert it with the included notebook or a one-liner:

```python
import pandas as pd
pd.read_csv("locations.csv").to_json("locations.json", orient="records", indent=2)
```

Save to the same folder as `map.html` and refresh the browser — no rebuild needed.

---

## 4. Interface overview

**Left sidebar (three tabs):**

- **Explore** — search box (matches name + type) with a dropdown of live results, plus a category list showing each type's color, icon, and count. Click a category to toggle it; use *Show all / Clear / Invert* for bulk actions.
- **Tools** — set your position via GPS or by clicking the map; find nearest resource with *Prev / Next* iteration; distance measurement tool (multi-segment path); fit-to-all-markers; export currently visible markers as JSON.
- **Stats** — total/visible/category counts, bounding-box area in km², a sorted bar chart of distribution by category, geographic centroid, mean pairwise distance, and point density.

**On the map:**

- Top-right: zoom controls, fullscreen toggle, theme toggle (dark/light).
- Bottom-right: basemap swatches (Dark, Light, Street, Satellite, Topo).
- Bottom-left: live cursor latitude/longitude and current zoom level.
- Bottom-center (when routing): route panel with drive/walk tabs, distance, duration, and straight-line distance.

---

## 5. Routing

Click any marker → **Route** in its popup. OSRM's public demo server calculates the driving route; walking uses a straight-line estimate at 4.5 km/h (OSRM's foot profile is unreliable on rural Garhwal roads).

**Off-road endpoints:** since OSRM snaps coordinates to the nearest road, and many POIs (temples, offices) sit off the road network, the app now draws thin dashed "last-mile" connectors from your actual origin/destination to the snapped road points. Distance and duration metrics include this off-road portion, with a sub-hint showing the split (e.g. `228 m road + 42 m off-road`). Off-road time is added at walking speed.

For production/research use, self-host OSRM with a locally-edited OSM extract that includes footpaths for better last-mile accuracy.

---

## 6. Measuring distances

Tools tab → *Distance / path measure*. Click on the map to add vertices, double-click to finish. The top-center panel shows segment count and cumulative distance. *Clear* wipes the path; *Done* exits measure mode. `Esc` also exits.

---

## 7. Keyboard shortcuts

| Key | Action |
|-----|--------|
| `/` | Focus search |
| `F` | Fit all markers |
| `M` | Toggle measurement tool |
| `L` | Use GPS location |
| `Esc` | Cancel pick/measure mode, clear search |

---

## 8. Exporting

Tools tab → *Export visible (JSON)* downloads whatever is currently visible (after category filters and search) as `visible_resources.json` in the same schema as the input.

---

## 9. Adding a new category

Edit the `CATEGORIES` object near the top of the `<script>` in `map.html`:

```js
const CATEGORIES = {
  temple: { label: "Temple", icon: "🛕", color: "#f59e0b" },
  // add your new type here:
  busstop: { label: "Bus Stop", icon: "🚏", color: "#0ea5e9" }
};
```

Then make sure the `type` field in `locations.json` uses the exact key (`"busstop"`). Unknown types are auto-registered with a grey pin, so this step is optional.

---

## 10. Known limitations

- **OSRM public demo server** is rate-limited and not for production traffic. For heavy use, spin up your own OSRM instance.
- **Walking routes** are straight-line estimates — they do not follow footpaths.
- **Off-road snap distance** can exceed 100 m for remote POIs (temples on trails, courtyards not in OSM). The dashed connector makes this visible but doesn't pathfind through it.
- **Mobile:** the sidebar overlays the map on narrow screens; tap the hamburger to toggle.
- **No offline support** — map tiles and OSRM require internet.

---

## 11. Modifying for other regions

Change the initial map center and zoom in `map.html`:

```js
const map = L.map("map", { zoomControl: false, attributionControl: true })
  .setView([30.5275, 79.095], 13);  // ← [lat, lon], zoom
```

Drop in a new `locations.json` and the sidebar, stats, and filters rebuild themselves.
