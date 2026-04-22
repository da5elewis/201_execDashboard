# Fast Forward Logistics Executive Dashboard - Brief

## Overview
A single page analytics dashboard highlighting KPIs, metrics, and trends to be used in leadership meetings to see how the business is running.

## Problem and Opportunity Definition
Fast Forward Logistics leadership currently relies on fragmented spreadsheets, emailed PDF reports, and ad-hoc data pulls to assess the health of the business before and during monthly leadership meetings. Financial KPIs (margin, accessorial recovery, buy-rate variance) live in the TMS and accounting systems, while operational KPIs (OTD, exception rate, dwell time) sit in dispatch and carrier management tools. Pulling these together takes hours of analyst time each month, often arrives stale, and provides no interactive drill-down capability.

**The opportunity** is a single-page, self-service dashboard that unifies financial and service reliability metrics into one view, updated monthly, that any executive can open in a browser and immediately understand where the business is winning and where it is bleeding margin or service quality — without waiting for an analyst to build a deck. By surfacing automated narrative insights (margin killers, service red flags, efficiency gaps), the dashboard shifts leadership conversations from "what happened?" to "what do we do about it?"

## User Persona
**Name:** Regional VP of Operations / CFO / SVP of Logistics  
**Role:** Senior leader at Fast Forward Logistics responsible for P&L performance, carrier strategy, or customer delivery commitments.  
**Context:** Attends monthly (or bi-weekly) leadership meetings where financial and operational performance is reviewed across regions (East, Central, West).  
**Goals:**
- See at a glance whether gross margin, net margin, and service levels are on target
- Compare current-month performance to the prior period and to the national average
- Identify the one or two issues that need immediate attention before the next meeting
- Drill into regional performance to understand whether problems are localized or systemic

**Pain points:**
- Currently waits 2–3 days after month-close for a static slide deck that can't be filtered
- Can't easily compare regions side by side or toggle between time periods
- Narrative context (why a number moved) is verbal and not captured alongside the data

**Tech comfort:** Comfortable with web-based tools; does not write SQL or build reports. Expects a polished, mobile-friendly experience they can check from a phone before a meeting.

## Business Case
1. **Time savings** — Eliminates 8–12 hours per month of manual report assembly by analysts, freeing them for deeper root-cause analysis.
2. **Faster decision-making** — Interactive filtering (by month, by region) lets leaders answer follow-up questions in real time during meetings rather than tabling them for the next cycle.
3. **Margin protection** — Automated "Top Margin Killer" and "Efficiency Gaps" insights surface revenue leakage (unrecovered accessorials, buy-rate overspend) that might otherwise go unnoticed for weeks.
4. **Service accountability** — Visible OTD, exception rate, and dwell-time trends create shared accountability across regions and make it harder for service issues to hide in averages.
5. **Scalability** — The architecture (Vue 3 + Vuetify + Chart.js consuming a structured JSON schema) is designed to swap in a live API or WebSocket feed when the business is ready to move from monthly to near-real-time data.

## Jobs to be Done
1. **When** I am preparing for a leadership meeting, **I want to** open the dashboard and see current-month KPIs with delta indicators, **so that** I know immediately what improved and what degraded since last month.
2. **When** a metric is flagged red, **I want to** toggle the region filter to East / Central / West, **so that** I can isolate whether the problem is regional or company-wide.
3. **When** I need to explain a trend to the board, **I want to** view a line chart of gross margin over the last 7+ months, **so that** I can show whether a dip is a one-time event or a sustained decline.
4. **When** the meeting discussion shifts to service quality, **I want to** see OTD and dwell-time trends on a bar chart alongside financial cards, **so that** I can correlate service failures with margin compression.
5. **When** I don't have time to analyze every number, **I want to** read the auto-generated Performance Snapshot insights, **so that** I get a plain-English summary of the biggest risks and can prioritize my attention.
6. **When** I am between meetings on my phone, **I want to** pull up the dashboard on a mobile browser, **so that** I can quickly check KPIs without needing my laptop.

## Functional Requirements
1. **KPI Summary Cards** — Display 5 financial cards (Gross Margin, Net Margin, Accessorial Recovery, Buy-Rate Variance, Revenue Quality RPM) and 4 service cards (OTD, Exception Rate, First-Time-Right, Dwell Time) with current value, target, and a color-coded delta arrow vs. the prior period.
2. **Period Picker** — A top-bar selector offering "All Months" per year and individual months (Jan 2024 – Apr 2026). Selecting a period filters all cards, charts, and insights to that scope.
3. **Region Picker** — A top-bar selector for National, East, Central, and West. When a region is selected, cards and charts reflect that region's data; cards also show the national value for comparison.
4. **Gross Margin Line Chart** — A scrollable 7-month window line chart of gross margin over time with navigation arrows and click-to-select-month interaction.
5. **Service Bar Chart** — A scrollable 7-month window bar chart showing OTD and Dwell Time over time, with the same navigation and click-to-select behavior as the margin chart.
6. **Performance Heatmap** — A regional comparison heatmap showing all KPIs across National, East, Central, and West for the selected period, color-coded against targets.
7. **Performance Snapshot** — Three auto-generated narrative insight cards (Top Margin Killer, Service Red Flag, Efficiency Gaps) derived from the data, surfacing the worst-performing months and their likely causes.
8. **Theme Toggle** — Light/dark theme switch in the app bar.
9. **Responsive Layout** — Cards stack vertically on mobile; charts remain usable on small screens via Vuetify's grid system (`v-container`, `v-row`, `v-col`).
10. **No Backend Dependency** — All data sourced from a local JSON file (`src/data/metrics.json`); no API calls required to render the dashboard.

## Set Up
- Create a new Vue project called "FastForward-Dashboard". Use Vite with Typescript and Vue Router. Then add Vuetify 3 with Material Design Icons and install chart.js and vue-chartjs for data visualizations.  Set everything up and open the project.
]

## Data
Generate a fake dataset as a JSON file (src/data/metrics.json).  28 months of data (Jan 2024-April 2026) Each month contains:
- Gross Margin % (percentage, vary monthly data from below, at, and  > target of 15%-18%)
- Net Margin % (percentage, vary monthly data, target is  >12%)
- Accessorial Recovery % (percentage, vary monthly data, target is  > 85%,)
- Buy-Rate Variance % (percentage, vary monthly data, target range +/- 5%)
- Revenue Quality RPM (dollar amount, vary monthly data, target range  >$2.50)
- On-Time Delivery OTD (percentage, vary monthly data, target >95%)
- Exception Rate (percentage, vary monthly data, target <3%)
- First-Time-Right (percentage, vary monthly data, target >90%)
- Dwell Time (Origin/Dest)	(number, vary monthly data ,target <2 hrs)

## Design and Layout (Vuetify)

### Header
- Black (`#000000`) header bar with Oswald H3 dashboard title and airplane icon left-aligned
- Region and Period selectors (equal width) top-right, stacked vertically on mobile (≤768px)

### Main Content
- Headline: "Financial Performance" followed by a row of 5 flat cards (Gross Margin, Net Margin, Accessorial Recovery, Buy-Rate Variance, Revenue Quality)
    - Each card: large value, delta arrow, target (small), region label, and (if not National) a vertical 1px pipe and national value for comparison
- Headline: "Service Reliability" followed by a row of 4 flat cards (On-Time Delivery, Exception Rate, First-Time-Right, Dwell Time)
    - Same card layout as above
- Two charts in a row:
    - Left: Gross Margin Over Time (headline styled like other sections, no card border)
    - Right: On-Time Delivery & Dwell Time (same headline, no card border)
- Below: "Performance Snapshot" section with three Material Icon insights (warning, flag, trending_down), each with a short narrative
- Below: Regional Performance Heatmap (no card border)

### Responsive & Style
- All layout uses Vuetify's v-container, v-row, v-col for grid
- Cards and controls stack vertically on mobile
- Clean, minimal, whitespace-rich, light theme only
- All cards and charts use flat style (no border/stroke)
- Consistent headline style for all sections

## Interactions
- Month picker in app bar filters EVERYTHING - summary cars show that month's numbers, charts highlight or filter to that month
- When "all" is selected summary cards show yearly totals/averages and charts show all 12 months
- Cards should small up/down arrow or color indicating change from previous month

## Style
- Light theme by default (Vuetify light theme)
- Clean, minimal, a lot of whitespace
- Charts us cohesive color pallette, natural colors
- Mobile reponsive - cards stack on small screens

## Tech
- Vue 3 + Typescript + Vuetify 3
- Chart.js via vue-chartjs for all charts
- Fake data from a local JSON file (no API calls)
- Single page - no routing needed for this app
