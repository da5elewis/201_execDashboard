<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { useTheme } from 'vuetify'
import metricsData from './data/metrics.json'
import GrossMarginChart from './components/GrossMarginChart.vue'
import ServiceChart from './components/ServiceChart.vue'
import PerformanceHeatmap from './components/PerformanceHeatmap.vue'

interface MonthData {
  month: string
  grossMargin: number
  netMargin: number
  accessorialRecovery: number
  buyRateVariance: number
  revenueQualityRPM: number
  onTimeDelivery: number
  exceptionRate: number
  firstTimeRight: number
  dwellTime: number
}

interface RawMonthData extends MonthData {
  regions: Record<string, Omit<MonthData, 'month'>>
}

const rawData = metricsData as RawMonthData[]

// Region picker
const selectedRegion = ref('National')
const regionOptions = ['National', 'East', 'Central', 'West']

// Project data based on selected region
const data = computed<MonthData[]>(() => {
  if (selectedRegion.value === 'National') {
    return rawData.map(({ regions, ...rest }) => rest)
  }
  return rawData.map(d => ({
    month: d.month,
    ...d.regions[selectedRegion.value]
  } as MonthData))
})

// Region data helper — get MonthData[] for a specific region
function regionDataFor(region: string): MonthData[] {
  return rawData.map(d => ({ month: d.month, ...d.regions[region] } as MonthData))
}

// Force light mode
const theme = useTheme()
theme.global.name.value = 'light'

// Month picker — year-level + individual months
const years = [...new Set(rawData.map(d => d.month.split('-')[0]))]

const monthOptions = computed(() => {
  const opts: { title: string; value: string }[] = []
  years.forEach(year => {
    opts.push({ title: `${year}: All Months`, value: `year-${year}` })
    rawData
      .filter(d => d.month.startsWith(year))
      .forEach(d => {
        const [y, m] = d.month.split('-')
        const label = new Date(+y, +m - 1).toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
        opts.push({ title: '  ' + label, value: d.month })
      })
  })
  return opts
})

const selectedMonth = ref('year-2026')

const isYearSelection = computed(() => selectedMonth.value.startsWith('year-'))
const selectedYear = computed(() => isYearSelection.value ? selectedMonth.value.replace('year-', '') : null)

// Chart time navigation — operates on full dataset
const chartOffset = ref(0)
const CHART_WINDOW = 7

// Initialize chart window based on current selection
function initChartOffset() {
  if (isYearSelection.value) {
    const yearStart = data.value.findIndex(d => d.month.startsWith(selectedYear.value!))
    chartOffset.value = yearStart >= 0 ? yearStart : 0
  } else {
    const idx = data.value.findIndex(d => d.month === selectedMonth.value)
    chartOffset.value = Math.max(0, idx - 6)
  }
}
initChartOffset()

const chartData = computed(() => {
  const start = Math.max(0, Math.min(chartOffset.value, data.value.length - CHART_WINDOW))
  const end = Math.min(data.value.length, start + CHART_WINDOW)
  return data.value.slice(start, end)
})

const canChartBack = computed(() => chartOffset.value > 0)
const canChartForward = computed(() => chartOffset.value + CHART_WINDOW < data.value.length)

function chartBack() {
  chartOffset.value = Math.max(0, chartOffset.value - 5)
}
function chartForward() {
  chartOffset.value = Math.min(data.value.length - CHART_WINDOW, chartOffset.value + 5)
}

function onChartSelectMonth(month: string) {
  selectedMonth.value = month
}

// Reset offset when filter changes
watch(selectedMonth, () => { initChartOffset() })
watch(selectedRegion, () => { initChartOffset() })

// Helper to average a subset of data
function avg(subset: MonthData[]): MonthData {
  const len = subset.length
  return {
    month: '',
    grossMargin: +(subset.reduce((s, d) => s + d.grossMargin, 0) / len).toFixed(1),
    netMargin: +(subset.reduce((s, d) => s + d.netMargin, 0) / len).toFixed(1),
    accessorialRecovery: +(subset.reduce((s, d) => s + d.accessorialRecovery, 0) / len).toFixed(1),
    buyRateVariance: +(subset.reduce((s, d) => s + d.buyRateVariance, 0) / len).toFixed(1),
    revenueQualityRPM: +(subset.reduce((s, d) => s + d.revenueQualityRPM, 0) / len).toFixed(2),
    onTimeDelivery: +(subset.reduce((s, d) => s + d.onTimeDelivery, 0) / len).toFixed(1),
    exceptionRate: +(subset.reduce((s, d) => s + d.exceptionRate, 0) / len).toFixed(1),
    firstTimeRight: +(subset.reduce((s, d) => s + d.firstTimeRight, 0) / len).toFixed(1),
    dwellTime: +(subset.reduce((s, d) => s + d.dwellTime, 0) / len).toFixed(1),
  }
}

// National data (always from top-level values)
const nationalData = computed<MonthData[]>(() => {
  return rawData.map(({ regions, ...rest }) => rest)
})

const nationalChartData = computed(() => {
  const start = Math.max(0, Math.min(chartOffset.value, nationalData.value.length - CHART_WINDOW))
  const end = Math.min(nationalData.value.length, start + CHART_WINDOW)
  return nationalData.value.slice(start, end)
})

const isRegionSelected = computed(() => selectedRegion.value !== 'National')
const showHeatmap = ref(false)

// Current data (for cards)
const currentData = computed(() => {
  if (isYearSelection.value) {
    const yearData = data.value.filter(d => d.month.startsWith(selectedYear.value!))
    return avg(yearData)
  }
  return data.value.find(d => d.month === selectedMonth.value)!
})

// National current data (for card comparison)
const nationalCurrentData = computed(() => {
  if (!isRegionSelected.value) return null
  if (isYearSelection.value) {
    const yearData = nationalData.value.filter(d => d.month.startsWith(selectedYear.value!))
    return avg(yearData)
  }
  return nationalData.value.find(d => d.month === selectedMonth.value)!
})

// Previous data (for delta arrows)
const previousData = computed((): MonthData | null => {
  if (isYearSelection.value) {
    // Compare to previous year
    const prevYear = String(+selectedYear.value! - 1)
    const prevYearData = data.value.filter(d => d.month.startsWith(prevYear))
    return prevYearData.length ? avg(prevYearData) : null
  }
  const idx = data.value.findIndex(d => d.month === selectedMonth.value)
  return idx > 0 ? data.value[idx - 1] : null
})

function delta(current: number, previous: number | undefined): { icon: string; color: string } | null {
  if (previous === undefined || previous === null) return null
  const diff = current - previous
  if (Math.abs(diff) < 0.05) return { icon: 'mdi-minus', color: 'grey' }
  return diff > 0
    ? { icon: 'mdi-arrow-up-bold', color: 'green' }
    : { icon: 'mdi-arrow-down-bold', color: 'red' }
}

function deltaReverse(current: number, previous: number | undefined): { icon: string; color: string } | null {
  if (previous === undefined || previous === null) return null
  const diff = current - previous
  if (Math.abs(diff) < 0.05) return { icon: 'mdi-minus', color: 'grey' }
  // lower is better for exception rate and dwell time
  return diff < 0
    ? { icon: 'mdi-arrow-down-bold', color: 'green' }
    : { icon: 'mdi-arrow-up-bold', color: 'red' }
}

// Metric card definitions
const financialCards = computed(() => {
  const c = currentData.value
  const p = previousData.value
  const n = nationalCurrentData.value
  return [
    { title: 'Gross Margin', value: c.grossMargin + '%', target: '15–18%', delta: delta(c.grossMargin, p?.grossMargin), nationalValue: n ? n.grossMargin + '%' : null, tooltip: 'Raw spread between what we bill the client and what we pay the carrier; the primary indicator of our basic pricing health.' },
    { title: 'Net Margin', value: c.netMargin + '%', target: '>12%', delta: delta(c.netMargin, p?.netMargin), nationalValue: n ? n.netMargin + '%' : null, tooltip: 'The actual profit remaining after accounting for \u201cleakage\u201d like cargo claims, unrecovered fees, and internal operational overhead.' },
    { title: 'Accessorial Recovery', value: c.accessorialRecovery + '%', target: '>85%', delta: delta(c.accessorialRecovery, p?.accessorialRecovery), nationalValue: n ? n.accessorialRecovery + '%' : null, tooltip: 'The percentage of \u201chidden\u201d costs (detention, lumper fees, layovers) billed by carriers that we successfully pass through to the customer.' },
    { title: 'Buy-Rate Variance', value: (c.buyRateVariance > 0 ? '+' : '') + c.buyRateVariance + '%', target: '±5%', delta: delta(c.buyRateVariance, p?.buyRateVariance), nationalValue: n ? (n.buyRateVariance > 0 ? '+' : '') + n.buyRateVariance + '%' : null, tooltip: 'The difference between our projected carrier cost (or market index) and what we actually paid; highlights if we are overpaying for capacity.' },
    { title: 'Revenue Quality (RPM)', value: '$' + c.revenueQualityRPM.toFixed(2), target: '>$2.50', delta: delta(c.revenueQualityRPM, p?.revenueQualityRPM), nationalValue: n ? '$' + n.revenueQualityRPM.toFixed(2) : null, tooltip: 'A measure of profitability per unit (e.g., Revenue per Mile); identifies if a high-volume client is actually worth the operational effort.' },
  ]
})

const serviceCards = computed(() => {
  const c = currentData.value
  const p = previousData.value
  const n = nationalCurrentData.value
  return [
    { title: 'On-Time Delivery (OTD)', value: c.onTimeDelivery + '%', target: '>95%', delta: delta(c.onTimeDelivery, p?.onTimeDelivery), nationalValue: n ? n.onTimeDelivery + '%' : null, tooltip: 'The percentage of shipments delivered within the customer\u2019s requested window; the ultimate benchmark for service reliability.' },
    { title: 'Exception Rate', value: c.exceptionRate + '%', target: '<3%', delta: deltaReverse(c.exceptionRate, p?.exceptionRate), nationalValue: n ? n.exceptionRate + '%' : null, tooltip: 'The frequency of shipments that encounter a \u201chiccup\u201d (damage, delays, or paperwork errors) requiring manual intervention by the ops team.' },
    { title: 'First-Time-Right', value: c.firstTimeRight + '%', target: '>90%', delta: delta(c.firstTimeRight, p?.firstTimeRight), nationalValue: n ? n.firstTimeRight + '%' : null, tooltip: 'The percentage of loads that move from tender to final invoice with zero manual corrections; the gold standard for operational efficiency.' },
    { title: 'Dwell Time (Origin/Dest)', value: c.dwellTime + ' hrs', target: '<2 hrs', delta: deltaReverse(c.dwellTime, p?.dwellTime), nationalValue: n ? n.dwellTime + ' hrs' : null, tooltip: 'The amount of time a carrier spends sitting at a shipper or receiver facility; high dwell times lead to detention costs and sour carrier relationships.' },
  ]
})

// Performance Snapshot insights
const insights = computed(() => {
  // Find worst months for each category
  const worstMargin = [...data.value].sort((a, b) => a.grossMargin - b.grossMargin)[0]
  const worstOTD = [...data.value].sort((a, b) => a.onTimeDelivery - b.onTimeDelivery)[0]
  const worstException = [...data.value].sort((a, b) => b.exceptionRate - a.exceptionRate)[0]
  const fmtMonth = (m: string) => {
    const [y, mo] = m.split('-')
    return new Date(+y, +mo - 1).toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
  }
  const iconColor = '#424242'
  return [
    {
      label: 'Top Margin Killer',
      icon: 'warning', // material icon name
      color: iconColor,
      text: `Gross margin hit a low of ${worstMargin.grossMargin}% in ${fmtMonth(worstMargin.month)} while accessorial recovery dropped to ${worstMargin.accessorialRecovery}%, indicating unrecovered fees are compressing margins.`,
    },
    {
      label: 'Service Red Flag',
      icon: 'flag', // material icon name
      color: iconColor,
      text: `OTD dropped to ${worstOTD.onTimeDelivery}% in ${fmtMonth(worstOTD.month)} due to carrier capacity shifts, forcing expensive backup capacity purchases and hurting margins.`,
    },
    {
      label: 'Efficiency Gaps',
      icon: 'trending_down', // material icon name
      color: iconColor,
      text: `Exception rate spiked to ${worstException.exceptionRate}% in ${fmtMonth(worstException.month)} with first-time-right at just ${worstException.firstTimeRight}%, signaling process breakdowns that increase handling costs and rework.`,
    },
  ]
})

// Heatmap rows — one per region + national, using current period filter
const heatmapRows = computed(() => {
  const regions = ['National', 'East', 'Central', 'West']
  return regions.map(region => {
    const source = region === 'National'
      ? nationalData.value
      : regionDataFor(region)
    let row: MonthData
    if (isYearSelection.value) {
      const filtered = source.filter(d => d.month.startsWith(selectedYear.value!))
      row = avg(filtered)
    } else {
      row = source.find(d => d.month === selectedMonth.value)!
    }
    return { region, ...row }
  })
})
</script>

<template>
  <v-app>
    <v-app-bar class="app-header" elevation="2" height="190">
      <div class="d-flex align-center ml-4">
        <v-icon class="header-icon">mdi-airplane</v-icon>
        <h3 class="header-title">Fast Forward Logistics</h3>
      </div>
      <div class="header-controls">
        <div class="header-selects">
          <v-select
            v-model="selectedRegion"
            :items="regionOptions"
            variant="outlined"
            density="compact"
            hide-details
            style="min-width: 180px; max-width: 180px;"
            label="Region"
            class="header-select"
          />
          <v-select
            v-model="selectedMonth"
            :items="monthOptions"
            item-title="title"
            item-value="value"
            variant="outlined"
            density="compact"
            hide-details
            style="min-width: 180px; max-width: 180px;"
            label="Period"
            class="header-select"
          />
        </div>
      </div>
    </v-app-bar>

    <v-main>
      <v-container fluid class="pa-6">
        <!-- Performance Snapshot -->
        <v-label class="text-h6 font-weight-bold text-grey-darken-2 mb-3">Performance Snapshot</v-label>
        <v-row>
          <v-col cols="12" md="8">
            <v-card variant="flat" rounded="lg">
              <v-list lines="three">
                <v-list-item
                  v-for="insight in insights"
                  :key="insight.label"
                >
                  <template #prepend>
                    <span class="material-icons insight-mat-icon" :style="{ color: insight.color }">{{ insight.icon }}</span>
                  </template>
                  <v-list-item-title class="font-weight-bold">{{ insight.label }}</v-list-item-title>
                  <v-list-item-subtitle class="text-wrap">{{ insight.text }}</v-list-item-subtitle>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>
          <v-col cols="12" md="4" class="d-flex align-center">
            <v-img
              src="/truck.png"
              alt="Fast Forward Logistics truck being unloaded"
              rounded="lg"
              cover
              max-height="260"
            />
          </v-col>
        </v-row>

        <!-- Financial Performance -->
        <v-label class="text-h6 font-weight-bold text-grey-darken-2 mt-8 mb-3">Financial Performance</v-label>
        <v-row>
          <v-col
            v-for="card in financialCards"
            :key="card.title"
            cols="12"
            sm="6"
            md="4"
            lg
          >
            <v-card variant="flat" class="text-center financial-card" rounded="lg" style="position: relative">
              <v-tooltip v-if="card.tooltip" location="top" max-width="280">
                <template #activator="{ props: tooltipProps }">
                  <v-icon v-bind="tooltipProps" size="14" color="grey" style="position: absolute; top: 8px; right: 8px; cursor: pointer">mdi-information-outline</v-icon>
                </template>
                {{ card.tooltip }}
              </v-tooltip>
              <v-card-item>
                <v-card-subtitle class="font-weight-medium card-label">{{ card.title }}</v-card-subtitle>
                <v-card-title class="d-flex align-center justify-center">
                  <template v-if="card.nationalValue">
                    <div class="d-flex align-center">
                      <div class="d-flex flex-column align-center">
                        <span class="text-h5 font-weight-bold">{{ card.value }}</span>
                        <span class="text-caption" style="color: #9E9E9E; line-height: 1; font-size: 0.5em">{{ selectedRegion }}</span>
                      </div>
                      <v-icon
                        v-if="card.delta"
                        :color="card.delta.color"
                        size="20"
                        class="ml-2"
                      >{{ card.delta.icon }}</v-icon>
                      <span class="vertical-pipe mx-2"></span>
                      <div class="d-flex flex-column align-center ml-1">
                        <span class="text-body-2" style="color: #9E9E9E">{{ card.nationalValue }}</span>
                        <span class="text-caption" style="color: #BDBDBD; line-height: 1; font-size: 0.5em">National</span>
                      </div>
                    </div>
                  </template>
                  <template v-else>
                    <span class="text-h5 font-weight-bold">{{ card.value }}</span>
                    <v-icon
                      v-if="card.delta"
                      :color="card.delta.color"
                      size="20"
                      class="ml-2"
                    >{{ card.delta.icon }}</v-icon>
                  </template>
                </v-card-title>
              </v-card-item>
              <v-card-text class="text-caption text-grey pt-0" style="font-size: 0.75em;">Target: {{ card.target }}</v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Service Reliability -->
        <v-label class="text-h6 font-weight-bold text-grey-darken-2 mt-8 mb-3">Service Reliability</v-label>
        <v-row>
          <v-col
            v-for="card in serviceCards"
            :key="card.title"
            cols="12"
            sm="6"
            md="3"
          >
            <v-card variant="flat" class="text-center financial-card" rounded="lg" style="position: relative">
              <v-tooltip v-if="card.tooltip" location="top" max-width="280">
                <template #activator="{ props: tooltipProps }">
                  <v-icon v-bind="tooltipProps" size="14" color="grey" style="position: absolute; top: 8px; right: 8px; cursor: pointer">mdi-information-outline</v-icon>
                </template>
                {{ card.tooltip }}
              </v-tooltip>
              <v-card-item>
                <v-card-subtitle class="font-weight-medium card-label">{{ card.title }}</v-card-subtitle>
                <v-card-title class="d-flex align-center justify-center">
                  <div v-if="card.nationalValue" class="d-flex flex-column align-center">
                    <span class="text-h5 font-weight-bold">{{ card.value }}</span>
                    <span class="text-caption" style="color: #9E9E9E; line-height: 1; font-size: 0.5em">{{ selectedRegion }}</span>
                  </div>
                  <span v-else class="text-h5 font-weight-bold">{{ card.value }}</span>
                  <v-icon
                    v-if="card.delta"
                    :color="card.delta.color"
                    size="20"
                    class="ml-2"
                  >{{ card.delta.icon }}</v-icon>
                  <div v-if="card.nationalValue" class="d-flex flex-column align-center ml-3">
                    <span class="text-body-2" style="color: #9E9E9E">{{ card.nationalValue }}</span>
                    <span class="text-caption" style="color: #BDBDBD; line-height: 1; font-size: 0.5em">National</span>
                  </div>
                </v-card-title>
              </v-card-item>
              <v-card-text class="text-caption text-grey pt-0" style="font-size: 0.75em;">Target: {{ card.target }}</v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Charts -->
        <v-row class="mt-8">
          <v-col cols="12" md="6">
            <v-card variant="outlined" rounded="lg">
              <v-card-item>
                <template #default>
                  <v-card-title class="text-subtitle-2 font-weight-bold text-grey-darken-2">Gross Margin Over Time</v-card-title>
                </template>
                <template #append>
                  <v-btn icon variant="text" size="small" :disabled="!canChartBack" @click="chartBack">
                    <v-icon>mdi-chevron-left</v-icon>
                  </v-btn>
                  <v-btn icon variant="text" size="small" :disabled="!canChartForward" @click="chartForward">
                    <v-icon>mdi-chevron-right</v-icon>
                  </v-btn>
                </template>
              </v-card-item>
              <v-card-text>
                <GrossMarginChart :data="chartData" :national-data="isRegionSelected ? nationalChartData : undefined" :highlighted-month="selectedMonth" @select-month="onChartSelectMonth" />
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card variant="outlined" rounded="lg">
              <v-card-item>
                <template #default>
                  <v-card-title class="text-subtitle-2 font-weight-bold text-grey-darken-2">On-Time Delivery & Dwell Time</v-card-title>
                </template>
                <template #append>
                  <v-btn icon variant="text" size="small" :disabled="!canChartBack" @click="chartBack">
                    <v-icon>mdi-chevron-left</v-icon>
                  </v-btn>
                  <v-btn icon variant="text" size="small" :disabled="!canChartForward" @click="chartForward">
                    <v-icon>mdi-chevron-right</v-icon>
                  </v-btn>
                </template>
              </v-card-item>
              <v-card-text>
                <ServiceChart :data="chartData" :national-data="isRegionSelected ? nationalChartData : undefined" :highlighted-month="selectedMonth" @select-month="onChartSelectMonth" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Performance Heatmap -->
        <template v-if="!isRegionSelected || showHeatmap">
          <v-label class="text-h6 font-weight-bold text-grey-darken-2 mt-8 mb-3">Regional Performance Heatmap</v-label>
          <v-card variant="outlined" rounded="lg">
            <v-card-text>
              <PerformanceHeatmap :rows="heatmapRows" />
            </v-card-text>
          </v-card>
        </template>

        <!-- Heatmap toggle (regional view only) -->
        <div v-if="isRegionSelected" class="d-flex justify-center mt-4">
          <v-btn
            variant="text"
            size="small"
            color="grey"
            @click="showHeatmap = !showHeatmap"
          >
            <v-icon class="mr-1" size="16">{{ showHeatmap ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
            {{ showHeatmap ? 'Hide' : 'Show' }} Regional Heatmap
          </v-btn>
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<style>
html, body {
  background: #FAFAFA;
}
.financial-card {
  background-color: #F5F5F5 !important;
  border-color: #BDBDBD !important;
}
.app-header {
  background-color: #9c0017 !important;
  color: #fff !important;
}
.header-icon {
  font-size: 64px !important;
  color: #fff;
  margin-right: 16px;
}
.header-title {
  font-family: 'Oswald', sans-serif;
  font-weight: 600;
  font-size: 1.6rem;
  color: #fff;
  margin: 0;
  letter-spacing: 0.5px;
}
.header-select .v-field {
  color: #fff !important;
}
.header-select .v-field__outline {
  --v-field-border-opacity: 0.4;
  color: #fff !important;
}
.header-select .v-label {
  color: rgba(255, 255, 255, 0.7) !important;
}
.header-select .v-select__selection-text,
.header-select .v-field__input {
  color: #fff !important;
}
.header-select .v-icon {
  color: #fff !important;
}
/* Header controls layout */
.header-controls {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  justify-content: flex-start;
  flex: 1;
  margin-right: 24px;
}
.header-toggle-row {
  width: 100%;
  display: flex;
  justify-content: flex-end;
}
.header-selects {
  display: flex;
  flex-direction: row;
  gap: 12px;
  margin-top: 8px;
}
.theme-toggle-btn {
  margin-left: 0;
  margin-top: 0;
}
@media (max-width: 768px) {
  .header-controls {
    flex-direction: column;
    align-items: flex-end;
    margin-right: 8px;
  }
  .header-selects {
    flex-direction: column;
    gap: 6px;
    margin-top: 8px;
  }
  .theme-toggle-btn {
    margin-left: 0;
    margin-top: 0;
  }
}
/* Material icon for insights */
.insight-mat-icon {
  font-family: 'Material Icons', 'Material Symbols Outlined', sans-serif;
  font-size: 28px;
  vertical-align: middle;
  margin-right: 28px;
}
.vertical-pipe {
  display: inline-block;
  width: 1px;
  height: 2.2em;
  background: #BDBDBD;
  margin: 0 12px;
  vertical-align: middle;
}
.card-label {
  margin-bottom: 10px;
}
</style>
