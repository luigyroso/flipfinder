# Flip Finder

Browser-based tool for finding apartment flip deals in Bogotá, Colombia. Scrapes listings from **Metrocuadrado** and **Finca Raíz** in real time — no backend required.

## Features

- Live search across MC and FR with configurable filters (price, area, rooms, bathrooms, parking, location)
- "Near me" proximity filter using browser geolocation
- NUEVO badges for newly discovered listings across searches
- Read/unread tracking
- Favorites, contacted, and discarded status workflow
- Notes per listing
- Google Street View integration
- Interactive map with listing markers
- All state persisted in `localStorage`

## Usage

Open `dashboard.html` in any modern browser. That's it — single file, no dependencies, no server.

## How It Works

- **Metrocuadrado**: Direct REST API calls (CORS-enabled from `file://`)
- **Finca Raíz**: Multi-strategy approach — tries internal API first, falls back to CORS proxy HTML scraping with `__NEXT_DATA__` parsing

## Files

| File | Description |
|------|-------------|
| `dashboard.html` | The entire app — HTML, CSS, and JS in one file |
| `NOTE.md` | Project decision log and task tracker |
| `decision-log.docx` | Early-stage decision documentation |
| `flip-finder-roadmap.md` | Feature roadmap |
