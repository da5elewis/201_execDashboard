# Fast Forward Logistics — Executive Dashboard

A single-page analytics dashboard for **Fast Forward Logistics** leadership meetings. Surfaces key financial and operational KPIs across 28 months of data (Jan 2024 – Apr 2026), making it easy to spot trends, compare periods, and identify performance issues at a glance.

---

## 🚀 Quick Start

1. **Install dependencies** (requires Node.js v18+):
   ```bash
   npm install
   ```
2. **Start the dev server:**
   ```bash
   npm run dev
   ```
   Visit [http://localhost:5173](http://localhost:5173) in your browser.
3. **Build for production:**
   ```bash
   npm run build
   ```
4. **Preview production build:**
   ```bash
   npm run preview
   ```

---

## 📸 Preview
![Dashboard Preview](/public/preview.png)

---

## 🧭 How It Works

- **Region & Period Pickers:** Top bar lets you filter all data by region (National, East, Central, West) and by month or year.
- **Performance Snapshot:** Auto-generated narrative insights highlight margin killers, service red flags, and efficiency gaps.
- **Financial & Service Cards:** Show current values, targets, and month-over-month/national deltas for each KPI.
- **Interactive Charts:** Gross Margin (line), On-Time Delivery & Dwell Time (dual-axis line), with scroll and click-to-filter.
- **Regional Heatmap:** Compare all KPIs across regions for the selected period.
- **Mobile-First Design:** Responsive layout using Vuetify 3 grid system.
- **Light/Dark Theme Toggle:** (if enabled in code)

---

## Key Features
- **Narrative Insights Engine**: Automatically flags "margin killers" and service red flags using a logic-based snapshot generator.
- **Bi-Directional Filtering**: Synchronized state management between the Period Picker and Chart.js instances.
- **Responsive Logistics Grid**: Mobile-first layout using Vuetify's layout system for on-the-go executive review.
- **Financial Performance cards** — Gross Margin, Net Margin, Accessorial Recovery, Buy-Rate Variance, and Revenue Quality (RPM) with month-over-month delta indicators
- **Service Reliability cards** — On-Time Delivery, Exception Rate, First-Time-Right %, and Dwell Time with color-coded trend arrows
- **Interactive charts** — Line chart for Gross Margin over time; bar chart for OTD & Dwell Time with scroll navigation and click-to-filter
- **Period picker** — Filter by individual month or view a full year; all cards and charts update in sync
- **Performance Snapshot** — Auto-generated narrative insights highlighting margin killers, service red flags, and efficiency gaps
- **Light / Dark theme toggle**
- **Responsive layout** — Cards stack on mobile screens via Vuetify's grid system

---

## 📊 Business KPIs
Financial Metrics
- Gross Margin: The raw spread between what we bill the client and what we pay the carrier; the primary indicator of our basic pricing health.
- Net Margin: The actual profit remaining after accounting for "leakage" like cargo claims, unrecovered fees, and internal operational overhead.
- Accessorial Recovery: The percentage of "hidden" costs (detention, lumper fees, layovers) billed by carriers that we successfully pass through to the customer.
- Buy-Rate Variance: The difference between our projected carrier cost (or market index) and what we actually paid; highlights if we are overpaying for capacity.
- Revenue Quality: A measure of profitability per unit (e.g., Revenue per Mile); identifies if a high-volume client is actually worth the operational effort.

Operational Metrics
- On-Time Delivery (OTD): The percentage of shipments delivered within the customer’s requested window; the ultimate benchmark for service reliability.
- Exception Rate: The frequency of shipments that encounter a "hiccup" (damage, delays, or paperwork errors) requiring manual intervention by the ops team.
- First-Time-Right: The percentage of loads that move from tender to final invoice with zero manual corrections; the gold standard for operational efficiency.
- Dwell Time: The amount of time a carrier spends sitting at a shipper or receiver facility; high dwell times lead to detention costs and sour carrier relationships.

---

## 🛠 Tech Stack
- **Core:** Vue 3 (Composition API) + TypeScript
- **UI/UX:** Vuetify 3 (Material Design 3)
- **Visualization:** Chart.js (via vue-chartjs)
- **State:** Local JSON Schema (mock RESTful API)
- **Build Tool:** Vite

---

## 📦 Dependencies
- `vue` — Core framework
- `vuetify` — Material Design UI components
- `chart.js` & `vue-chartjs` — Data visualization
- `@mdi/font` — Material Design Icons
- `vite` — Fast dev/build tool
- `typescript` — Type safety

---

## 🛠 Customization
- **Update Data:** Edit `src/data/metrics.json` to change dashboard metrics (structure matches the displayed KPIs).
- **Add Metrics:** Add new fields to `metrics.json` and update chart/card logic in `src/components/` as needed.
- **Connect to API:** Replace the data import in `src/App.vue` with a real API call for live data.

---

## Roadmap
- [ ] Enable export to PDF or other shareable document
- [ ] Assess design against A11y Standards and refine as needed for improved accessibility
- [ ] Transition to real-time WebSockets for "Live Fleet" tracking.

---

## Getting Started (Detailed)
1. **Clone or download this repo.**
2. **Install dependencies:** `npm install`
3. **Run locally:** `npm run dev` (default: [http://localhost:5173](http://localhost:5173))
4. **Edit data:** All dashboard data is in `src/data/metrics.json` (no backend required).
5. **Build for production:** `npm run build`
6. **Preview production build:** `npm run preview`

---

## 🏗 Project Structure
- `src/components/`: Modularized chart components (GrossMarginChart, ServiceChart, PerformanceHeatmap, etc.)
- `src/data/`: Structured JSON mimicking a relational database output
- `public/`: Static assets (preview images, icons, logo)

```
src/
├── App.vue                  # Main layout, state, and card/insight logic
├── main.ts                  # App entry point
├── style.css                # Global styles
├── components/
│   ├── GrossMarginChart.vue # Line chart — Gross Margin over time
│   ├── ServiceChart.vue     # Dual-axis chart — OTD & Dwell Time
│   ├── PerformanceHeatmap.vue # Regional KPI heatmap
│   └── HelloWorld.vue       # (Starter/demo component)
└── data/
    └── metrics.json         # 28 months of generated KPI data
public/
├── preview.png              # Dashboard screenshot
├── FFL_Logo.png             # Company logo
├── icons.svg, favicon.svg   # Icons
├── truck.png                # Illustration for snapshot
```

---

## 🖼 More Screenshots
![Mobile Preview](/public/mobile.png)
![Heatmap Preview](/public/heatmap.png)

---

## ❓ Troubleshooting & FAQ
- **Vuetify theme errors:** Ensure you are using Node.js v18+ and a compatible npm version.
- **Port already in use:** Change the port in `vite.config.ts` or stop the other process.
- **Data not updating:** Make sure you save changes to `src/data/metrics.json` and reload the page.
- **Build errors:** Run `npm install` to ensure all dependencies are installed.

---
