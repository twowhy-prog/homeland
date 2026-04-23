# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**RELO** (이사 후보지 & 재무 시뮬레이터) — a client-side SPA for Seoul apartment relocation planning. Users search apartment transactions, visualize geospatial proximity to points of interest, and run financial simulations for buying/selling.

## Running the App

No build step. Open `index.html` directly in a browser, or serve it with any static file server:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

There are no tests, no linter, and no package manager.

## Architecture

The entire application lives in **`index.html`** (~1,650 lines) with three sections: `<style>` (CSS), `<body>` (HTML), and a `<script>` block (all JavaScript). There is no bundler, no framework, and no server-side code.

### External Libraries (loaded via CDN)

- **Leaflet.js** v1.9.4 — interactive map
- **Turf.js** v6.5.0 — geospatial intersection calculations
- **Google Fonts** — Noto Sans KR, Bebas Neue, DM Mono

### External APIs

- **Korean Government Real Estate API** (`apis.data.go.kr/1613000/RTMSDataSvcApt*`) — apartment buy (`AptTrade`) and rent (`AptRent`) transaction data, paginated XML responses
- **CoinGecko API** — XRP and BTC spot prices, fetched through `https://api.allorigins.win/raw?url=` CORS proxy (with `corsproxy.io` as fallback)
- API service key is hardcoded in the script as `SERVICE_KEY`

### Core Modules (all inside `<script>`)

| Module | Key functions | Responsibility |
|---|---|---|
| **Map & Geospatial** | `initMap()`, `updateRadius()`, `updateIntersection()`, `renderMarkers()` | Leaflet map, 3 POI circles, Turf.js intersection polygon, apartment markers |
| **Data Loading** | `loadApts()`, `parseXML()`, `groupApts()` | Fetches 6 months × 5 LAWD codes × 2 types (60 requests in chunks of 6), parses XML, groups by apartment+dong+type |
| **Filtering & List** | `filtered()`, `renderList()`, `focusApt()` | Client-side filtering by price/area/intersection, renders up to 80 results |
| **Financial Simulator** | `buildFinance()`, `calcFinance()`, `calcTax()`, `calcAcqTax()`, `fetchCrypto()` | Capital gains tax (양도세), acquisition tax (취득세), long-term holding deductions, crypto portfolio, jeonse vs. purchase scenarios |
| **UI & Interactions** | `switchTab()`, `toggleBS()`, `showAptDetail()` | Tab switching, mobile bottom-sheet swipe gestures, apartment detail panel |

### State Management

- **`localStorage`** — persists financial inputs (`fin_*` keys) and POI radius values (`radius_*` keys) across sessions
- **Global variables** — `allGrouped` (all loaded apartments), `filters` (active price/area range), `POIS` (POI coordinates + radius), `tradeType` (`'buy'`/`'rent'`), `fin` (finance simulator state)

### Key Constants

```js
LAWD codes (5 Seoul districts): 11650, 11680, 11200, 11710, 11560
NBHD target neighborhoods: ['잠원동','반포동','서초동','압구정동','논현동','신사동']
PYEONG = 3.3058  // ㎡ per pyeong (평)
// Prices in 만원 (10,000 KRW), areas in 평
// Only 2026+ transactions are retained after grouping
```

### Responsive Layout

- **Desktop**: fixed sidebar (filter + list) + full-height map/finance panel side-by-side
- **Mobile (≤768px)**: full-screen map with a swipeable bottom sheet; tab bar switches between map markers and the finance simulator
