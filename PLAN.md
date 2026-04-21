# Fast Forward Logistics Executive Dashboard - Brief

## Overview
A single page analytics dashboard highlighting KPIs, metrics, and trends to be used in leadership meetings to see how the business is running.

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

## Layout (Vuetify)
- v-app-bar at the top with dashboard title and month picker
- month picker defaults to showing ALL months
- When specific month slected all cards and charts filter to that month. when "all" is selected, show full year
- below app bar: Headline "Financial Performance" and row of 5 summary cards (v-card) showing key metrics - Gross Margin, Net Margin, Accessorial Recovery Rate, Buy-Rate Variance, Revenue Quality (RPM) 
 below Financial Performance card row: Headline "Service Reliability" and row of 4 summary cards (v-card) showing key metrics - On-Time Delivery (OTD), Exception Rate, First-Time-Right %, Dwell Time (Origin/Dest)
- below the cards: row of 2 charts
    - Left: line chart showing gross margin over time
    - Right: bar chart showing On-Time Delivery and Dwell Time over time
- below that: one full-width list of insights Labeled "Performance Snapshot" including Top Margin Killer, Service Red Flag, Efficiency Gaps, each with an insight (example "Service Red Flag: OTD has dropped to 84% due to carrier capacity shifts, which is forcing us to buy expensive backup capacity, hurting margins.")
- use v-container, v-row, v-col for reponsive grid layout

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
