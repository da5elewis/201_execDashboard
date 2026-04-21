# Fast Forward Logistics — Executive Dashboard

A single-page analytics dashboard for **Fast Forward Logistics** leadership meetings. It surfaces key financial and operational KPIs across 28 months of data (Jan 2024 – Apr 2026), making it easy to spot trends, compare periods, and identify performance issues at a glance.

## Features

- **Financial Performance cards** — Gross Margin, Net Margin, Accessorial Recovery, Buy-Rate Variance, and Revenue Quality (RPM) with month-over-month delta indicators
- **Service Reliability cards** — On-Time Delivery, Exception Rate, First-Time-Right %, and Dwell Time with color-coded trend arrows
- **Interactive charts** — Line chart for Gross Margin over time; bar chart for OTD & Dwell Time with scroll navigation and click-to-filter
- **Period picker** — Filter by individual month or view a full year; all cards and charts update in sync
- **Performance Snapshot** — Auto-generated narrative insights highlighting margin killers, service red flags, and efficiency gaps
- **Light / Dark theme toggle**
- **Responsive layout** — Cards stack on mobile screens via Vuetify's grid system

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Vue 3 (Composition API, `<script setup>`) |
| Language | TypeScript |
| UI Library | Vuetify 4 + Material Design Icons |
| Charts | Chart.js via vue-chartjs |
| Build Tool | Vite |
| Data | Local JSON file (`src/data/metrics.json`) — no API calls |

## Getting Started

```bash
# Install dependencies
npm install

# Start the dev server (http://localhost:5173)
npm run dev

# Build for production
npm run build

# Preview the production build
npm run preview
```

## Project Structure

```
src/
├── App.vue                  # Main layout, state, and card/insight logic
├── main.ts                  # App entry point
├── style.css                # Global styles
├── components/
│   ├── GrossMarginChart.vue # Line chart — Gross Margin over time
│   └── ServiceChart.vue     # Bar chart — OTD & Dwell Time over time
└── data/
    └── metrics.json         # 28 months of generated KPI data
```