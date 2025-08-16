# Transit Equity and Food Access in San Diego

commands to run:
npm run build
npm run deploy

link -> https://wmcornejo.github.io/sdiego_transit/
**Goal**: Identify and visualize census tracts in San Diego that meet USDA’s food desert criteria *and* are underserved by public transit.

---

## Data:

1. **San Diego Grocery Stores**

   * Filter by stores with >\$2M in sales and all food departments (if possible)
   * Use DataSD, Yelp fusion API, or USDA SNAP retailer locator (for EBT-accepting stores)

2. **San Diego Census Tracts**

   * Get from [TIGER/Line shapefiles](https://www.census.gov/geographies/mapping-files/time-series/geo/tiger-line-file.html) or [Data.gov](https://data.gov)

3. **Demographic Attributes**

   * From the **American Community Survey (ACS)** via [Census API](https://www.census.gov/data/developers/data-sets/acs-5year.html)
   * Get:

     * Poverty rate
     * Median household income
     * % of households without vehicles

4. **Transit Stops / Routes**

   * GTFS data from [MTS](https://www.sdmts.com/business-center/doing-business-mts/developer-resources) or DataSD
   * Optionally use [OpenTripPlanner](https://www.opentripplanner.org/) or other routing APIs to compute walk-time

---

## WebGIS Features to build:

| Layer                 | Visualized As                     | Notes                                       |
| --------------------- | --------------------------------- | ------------------------------------------- |
| Census Tracts         | Choropleth map                    | Color based on income, poverty, or no-car % |
| Grocery Store Buffers | Walk polygons (0.25, 0.5, 1 mile) | Use Turf.js or network analysis tools       |
| Transit Stops         | Red dots or custom icons          | Cluster if dense                            |
| Overlay Tool          | Toggle layers on/off              | Vue or React component                      |
| Popup Tooltips        | Tract stats + links               | Bind popups to tracts or stores             |
| Legend + Scoring      | Color ramps for equity scores     | Combine raster-style scoring logic          |

---

## How to Build It: Stack Overview

| Component           | Tools                                                             |
| ------------------- | ----------------------------------------------------------------- |
| **Framework**       | Vue.js or React (you're using Vue already)                     |
| **Map Library**     | Leaflet (or Esri JS if you're targeting enterprise GIS jobs)      |
| **Geo Analysis**    | Turf.js (for distance, buffer, union), optionally PostGIS offline |
| **Data Processing** | Preprocess in QGIS, or use GeoJSON + reclass in code              |
| **Visualization**   | Color scales (D3, chroma.js), Leaflet choropleths                 |
| **Deployment**      | GitHub Pages, Netlify, or Vercel                                  |
| **(Optional)**      | Host your own API with Node.js if you want to fetch live data     |

---

## Scoring System:

You can create a **composite index (0–4)** for each census tract:

| Factor              | Range                          | Scoring Example                  |
| ------------------- | ------------------------------ | -------------------------------- |
| Median Income       | Quantile                       | 0–4 (low = 0)                    |
| Poverty Rate        | >20% = low-income              | 0–4                              |
| Distance to Grocery | via buffer                     | 0–4 (close = high score)         |
| Vehicle Ownership   | % no car                       | 0–4 (more no car = lower access) |
| Transit Access      | Transit stop within 0.25–0.5mi | 0/1 boolean or 0–4 range         |

> Final map can sum scores and color tracts: Green = good access, Red = underserved

---

## 🌍 Project Concept Example: “**Can You Get Groceries Without a Car in San Diego?**”

### Page layout:

* Search by address or zoom
* View all grocery stores + transit stops
* Heatmap or tract coloring based on food access equity
* Popup with tract-level stats
* Optional: Add "Create Your Own Score" sliders

---
