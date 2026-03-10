# Flip Finder — Project Notes

> Bogotá apartment flipping deal-flow tool. Single HTML file, runs entirely in the browser from `file://` origin. Resume point for all sessions related to this project. Read this first, update as you go.

---

## 1. Decisions

### Architecture
- **Single HTML file** — zero dependencies, no server, no build step. Everything (HTML, CSS, JS) in one self-contained file. Runs from `file://` in any browser.
- **Browser-only scraping** — the Metrocuadrado REST API only returns real data from `file://` origin (CORS restriction). No backend is needed or possible.
- **localStorage for persistence** — listings, statuses, favorites, and config all stored client-side. Keys: `ff_listings`, `ff_state`, `ff_config`, `ff_last_search`.

### Data Sources
- **Finca Raíz** — scrapes via CORS proxies, parses `__NEXT_DATA__` JSON from page HTML.
- **Metrocuadrado (MC)** — direct API call with `x-api-key` header. No CORS proxy needed.
- **Ciencuadras** — **disabled**. Blocks all three CORS proxies. Their API requires OAuth credentials (client_id/secret). No browser-only workaround exists.

### MC API Strategy
- **Endpoint:** `https://www.metrocuadrado.com/rest-search/search`
- **Auth:** `x-api-key: P1MfFHfQMOtL16Zpg36NcntJYCLFm8FqFfudnavl`
- **Critical discovery:** Real listings are in `data.recommended`, not `data.results`. The `results` array is either empty or contains only metadata like `{type:"tags"}`. The code checks `recommended` first.
- **Server-side filters work** — confirmed via `totalHits` comparison. Params: `saleRange=min,max`, `areaRange=min,max`, `roomList=2,3`, `stratumList=3,4`. Comma-separated format.
- **`areaRange` min not enforced server-side** — the API ignores the minimum area filter. Client-side filtering catches listings below `amin`.
- **Probe strategy** — try filtered first; if `totalHits > 0` and items returned, use filtered mode (up to 20 pages). Fall back to unfiltered (10 pages) if filtered returns nothing.
- **Pagination cap** — `Math.ceil(filteredHits / 50)` pages, capped at `min(c.pages, totalPages, 20)`.

### MC Item Field Names (from API response)
```
title         → listing title
link          → relative URL (e.g. /inmueble/...)
mvalorventa   → sale price (number, e.g. 260000000)
marea         → area in m² (number, e.g. 43.95)
mareac        → constructed area (same as marea usually)
mnrocuartos   → rooms (string, e.g. "3")
mnrobanos     → bathrooms (string, e.g. "1")
mnrogarajes   → garages (string)
midinmueble   → property code (e.g. "10858-M6371583")
mbarrio       → neighborhood (uppercase)
mnombrecomunbarrio → neighborhood (proper case)
mzona.nombre  → zone (Norte, Sur, etc.)
imageLink     → thumbnail URL
```
No estrato field in search results — requires separate enrichment via detail page.

### Proximity / Location Data
- MC API returns lat/lon in `recommended[]` items as `localizacion.lat` and `localizacion.lon` (e.g. `4.753, -74.028` for Bogota). Detected via keyword scan in `parseMCItem` — the flattened key `localizacion.lat` matches the `.endsWith(".lat")` rule.
- FR `__NEXT_DATA__` may contain coordinates in `it.coordinates`, `it.location`, `it.locations.location_point`, or `it.geoLocation`.
- Haversine formula used for distance calculation (km).
- Browser Geolocation API (`getCurrentPosition`) for user location; manual lat/lon fallback if denied.
- Listings without coordinates pass through the proximity filter (not hidden).

### Estrato Enrichment
- After search, enrich top listings for estrato via detail API or HTML scraping.
- 4 extraction strategies: ld+json, RSC flight data, plain text match, meta/data attributes.
- Estrato filter is **lenient**: listings with unknown estrato (0) pass through to avoid hiding valid results.

### CORS Proxies (fallback for Finca Raíz)
```
corsproxy.io    → https://corsproxy.io/?{url}
allorigins.win  → https://api.allorigins.win/raw?url={url}
codetabs.com    → https://api.codetabs.com/v1/proxy?quest={url}
```

### Search Filters (defaults)
| Filter | Default |
|--------|---------|
| City | Bogotá |
| Type | Apartamento |
| Offer | Venta |
| Price min | $150,000,000 COP |
| Price max | $300,000,000 COP |
| Area min | 55 m² |
| Area max | 80 m² |
| Rooms min | 2 |
| Rooms max | 4 |
| Estratos | 3, 4 (selectable chips) |

### Listing Status Model
- **Status** (mutually exclusive): `new`, `contacted`, `discarded`
- **Favorite** (independent boolean): can be set regardless of status
- Status + favorite survive across searches via `ff_state` localStorage key
- Re-searching keeps `contacted`, `discarded`, and `favorited` listings; replaces all `new` ones

---

## 2. Key Insights

- **MC API is the gold standard** — it's CORS-enabled from `file://`, has server-side filters, returns 50 items per page, and covers the widest inventory. Finca Raíz scraping is noisier and slower.
- **Results jump from ~28 to ~493** once server-side MC filters are applied (estratos 3+4, $150–300M, 55–80m², 2–3 rooms). Before filtering we were crawling unfiltered pages with client-side narrowing only.
- **`data.recommended` vs `data.results`** — all visible listings on MC are in `recommended`. This was the root cause of every listing showing "Ver en MC" / 0m² (Strategy 2 slug fallback was being triggered because Strategy 1 found no parseable items).
- **Ciencuadras is a dead end** for browser-only tools. All proxy routes blocked; OAuth required.
- **Estrato is not in MC search results** — it has to be enriched post-search from detail pages. This is slow (one request per listing), so only a sample gets enriched.
- **Area minimum isn't enforced server-side** — `areaRange` on the MC API only enforces the max. The min is silently ignored. ~132 sub-55m² listings slip through per search run.

---

## 3. Tasks Done (chronological)

1. Built initial Flip Finder single-HTML tool — FR + MC + CC scraping, search modal, listing cards, detail panel, status tracking (new/contacted/discarded), localStorage persistence.
2. Diagnosed Ciencuadras as unreachable (all proxies blocked, OAuth required). Disabled cleanly with console log and comment explaining why.
3. Investigated MC's low result count — discovered server-side API filter parameters via RSC flight data analysis on MC's website.
4. Confirmed filter param format: comma-separated (`saleRange=150000000,300000000`, etc.).
5. Implemented server-side filtered MC crawl with `totalHits`-based probe to detect whether filters are actually working. Results went from 28 → 493.
6. Discovered the real root cause of "Ver en MC" / 0m² cards: API listings are in `data.recommended`, not `data.results`. Fixed array key detection to check `recommended` first.
7. Optimized probe: skip redundant unfiltered call — go filtered-first, fall back only if 0 hits.
8. Fixed pagination early-exit: use `break` instead of `continue` when page returns 0 items.
9. Added `first_seen` / `last_seen` date tracking on all listings.
10. Added `ff_last_search` timestamp, "Última búsqueda" in header, and new-since-last-search count in toast.
11. Added daily search reminder banner — appears when >12h since last search, shows last search time, links to "Buscar ahora". Dismissable per session. Auto-hides after a search completes.
12. Fixed reminder banner not refreshing after search — added `checkReminder()` call inside `render()`.
13. Added favorites system — star icon on each card, Favorito button in detail panel, ★ Favoritos filter pill, header stat counter, keyboard shortcut `f`, persisted in localStorage, survives re-searches.
14. Fixed header stat counts not matching list when display filters active — stats now count from the display-filtered pool when "Filtrar" is on.
15. Added proximity filter ("Cerca de mí") — uses browser Geolocation API or manual lat/lon input. Haversine distance calculation, configurable 1–20km radius slider, auto-sorts by distance when activated. Distance shown on cards and detail panel. lat/lon extracted from MC API (`localizacion.lat`/`localizacion.lon`) and FR __NEXT_DATA__, persisted through merge/localStorage.
16. Fixed `_mcFieldDump` being reset at end of search — now resets at start of search instead so it can be inspected in console after search completes.
17. Separated NUEVO badge and highlight into two independent features: (a) **NUEVO badge** = generation-based (`first_seen_gen === searchGen`). Shows only on first discovery; next search removes it with no replacement label. (b) **Highlight** = click-based (`!l.read`). Blue background + blue left border stays until user clicks the card. These are orthogonal — a listing can be NUEVO+highlighted, NUEVO+not-highlighted (clicked same search), not-NUEVO+highlighted (old but unclicked), or neither. Selected card uses gray style to be visually distinct from unread blue. `read` persists in `ff_state.rd`. Verified with 8-assertion automated test.
18. Fixed favorite star icon overlapping the status badge — moved star down from `top:10px` to `top:32px`.
19. Auto-enable display filters after every search — `displayFiltersOn` is set to `true` at the end of `runSearch()` so results are always filtered without needing to click the button.
20. Fixed critical NUEVO badge bug — `runSearch()` filters out all "new" status listings before calling `merge()`, so returning listings were never found in `ex` and always got a fresh `first_seen_gen`, making everything appear NUEVO. Fix: `merge()` now also checks persisted `ff_state` (via `loadSt()`) as a fallback for items that were filtered out. Also fixed JS falsy-zero bugs where `first_seen_gen === 0` was treated as "not set" by `||` operators — replaced with explicit `!= null` checks. Initialized `searchGen` base to 1 to avoid gen-0 edge cases.
21. Added `garages` (parqueadero) field to all parsers: MC API auto-detects fields with garage/garaje/parking/parqueadero keywords, MC slug parses N-garajes pattern, FR reads `garages`/`parking`/`parkingLots`/`garajes`/`parqueaderos` fields. Added baños (min/max) and parqueadero (min/max) filter inputs to search modal. Both filters apply in display filtering (`getFiltered`, `getFilteredBase`) and diagnostics. Cards show "N parq" when garages > 0; detail panel shows "N parqueadero(s)" tag.
22. Improved selected card highlight in sidebar — changed from subtle gray to a blue left border (4px) with inset blue-light box-shadow, making the active card clearly distinguishable from both unread (full blue bg) and normal cards.
23. Added Google Street View button in detail panel actions — opens `maps.google.com` with `map_action=pano&viewpoint=LAT,LON`. Only shown when listing has lat/lon coordinates.
24. Added phone/contact extraction across all parsers (background — extraction code kept but UI only shows when phones are actually found, which is rare since MC/FR hide contacts behind JS buttons). Removed "Contacto: no disponible" fallback, card phone display, and Reset button from detail panel to reduce clutter. Phones persisted in `ff_state.ph`.
25. Added notes section for favorited/contacted/discarded listings. Textarea appears in detail panel between actions and images. Auto-saves on input. Persisted in `ff_state.nt` and preserved through merge/applySt.
26. Overhauled FR crawler with multi-strategy approach: (1) Tries FR internal API (`api.fincaraiz.com.co/document/api/...`) with POST body filters — this is the mobile/internal API that may or may not be CORS-enabled. (2) Falls back to CORS proxy HTML fetch with `__NEXT_DATA__` parsing. Added 2 more CORS proxies (`corsproxy.org`, `thingproxy.freeboard.io`). FR now crawls `pages*2` (up to 10) to compensate for fewer items/page vs MC. Added comprehensive logging (`[FR]` prefix) so proxy failures and empty results are visible in console. Early stop if page 1 fails or a page returns 0 items.

---

## 4. Tasks Pending

- **Scheduled reminder**: No persistent scheduler set up. The VM is ephemeral; systemd timers don't survive sessions. The reminder banner inside the dashboard is the working alternative (triggers on open if >12h since last search).
- **Ciencuadras**: Could be added if the user gets OAuth credentials or a server-side proxy. The `parseCiencuadras` function is already written and in the file — just disabled.
- **Estrato enrichment coverage**: Currently enriches a limited sample. Could be expanded or made configurable.
- **Export**: No way to export listings to CSV/spreadsheet yet.
- **FR coverage**: FR has no public API and relies on CORS proxies which are inherently unreliable. The `api.fincaraiz.com.co` internal API probe is speculative. If FR results remain low, the user should check the browser console for `[FR]` logs to see which strategy/proxy is working. A server-side proxy would be the definitive fix.
- **Notes field**: No free-text notes per listing.
- **Publication date**: Investigated — MC search API does not return any publish date field. Dropped as a feature. `first_seen` + generation-based NUEVO badge serve as the proxy for discovery freshness.
- **Proximity coverage**: lat/lon extraction depends on what the APIs return. MC API likely has coordinates for most listings; FR coverage unknown until tested live. Listings without coords are shown with no distance badge rather than hidden.

---

## 5. Assets

### File
```
dashboard.html  — The full tool. Open in browser from file:// (no server needed).
```
Location when last saved: your Cowork outputs folder.

### MC API Quick Reference
```
Endpoint:   GET https://www.metrocuadrado.com/rest-search/search
Header:     x-api-key: P1MfFHfQMOtL16Zpg36NcntJYCLFm8FqFfudnavl
Params:
  realEstateTypeList=apartamento
  realEstateBusinessList=venta
  city=bogota%20d.c.
  from=0          (pagination offset, increments by 50)
  size=50
  saleRange=150000000,300000000
  areaRange=55,80
  roomList=2,3
  stratumList=3,4

Response shape:
  data.totalHits        → total matching listings
  data.recommended[]    → actual listing objects (NOT data.results)
  data.results[]        → empty or metadata only
```

### localStorage Keys
```
ff_listings     → full listings array (JSON)
ff_state        → status/favorite/first_seen per listing ID (JSON)
ff_config       → search config (city, filters, etc.)
ff_last_search  → ISO timestamp of last completed search
ff_search_gen   → integer, incremented on each search run
```

### `ff_state` Entry Shape
```json
{
  "<listing_id>": {
    "s":  "new" | "contacted" | "discarded",
    "t":  "2025-03-06T14:30:00.000Z",  // status changed at
    "fs": "2025-03-04T09:00:00.000Z",  // first seen
    "fv": true,                         // favorite
    "fg": 3,                            // first seen generation (search run #)
    "rd": true,                         // read (clicked by user)
    "ph": ["3001234567"],               // contact phone numbers
    "nt": "Llamar mañana"              // user notes
  }
}
```

### Keyboard Shortcuts
```
f   → toggle favorite on selected listing
c   → mark as contacted
d   → discard
r   → reset to new
```

### CORS Proxy Chain (for Finca Raíz)
```javascript
const PROXIES = [
  u => `https://corsproxy.io/?${encodeURIComponent(u)}`,
  u => `https://api.allorigins.win/raw?url=${encodeURIComponent(u)}`,
  u => `https://api.codetabs.com/v1/proxy?quest=${encodeURIComponent(u)}`,
];
// cFetch(url, timeout) tries each proxy in order, returns first successful response
```

### parseMCItem — Key Field Detection Logic
```javascript
// Price:  key includes "valor" AND value > 1e6  → mvalorventa
// Area:   key includes "area" AND value 5–10000 → marea / mareac
// Rooms:  key includes "cuart" AND value 1–30   → mnrocuartos (string parsed via parseFloat)
// Baths:  key includes "bano" AND value 1–30    → mnrobanos
// URL:    key includes "link"                   → link field
// Hood:   key includes "barrio"                 → mbarrio / mnombrecomunbarrio
// Code:   it.midinmueble || it.propertyCode     → for estrato enrichment lookups
```

---

*Last updated: March 9, 2026*