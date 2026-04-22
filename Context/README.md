# Fast Forward Logistics — Executive Dashboard

A single-page analytics dashboard for **Fast Forward Logistics** leadership meetings. It surfaces key financial and operational KPIs across 28 months of data (Jan 2024 – Apr 2026), making it easy to spot trends, compare periods, and identify performance issues at a glance.

## 📸 Preview
![Dashboard Preview](/public/preview.png)


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

## Business KPIs
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

## 🛠 Tech Stack
- **Core**: Vue 3 (Composition API) + TypeScript
- **UI/UX**: Vuetify 3 (Material Design 3)
- **Visualization**: Chart.js (via vue-chartjs)
- **State**: Local JSON Schema (Mocking a RESTful API structure)


## Roadmap
- [ ] Enable export to PDF or other shareable document
- [ ] Assess design against A11y Standards and refine as needed for improved accessibility
- [ ] Transition to real-time WebSockets for "Live Fleet" tracking.

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
- To update the dashboard data, modify src/data/metrics.json following the existing schema.

## 🏗 Project Structure
- `src/components/`: Modularized chart components for high reusability.
- `src/data/`: Structured JSON mimicking a relational database output.

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