<script setup lang="ts">
import { ref, computed } from 'vue'
import metricsData from './data/metrics.json'
import GrossMarginChart from './components/GrossMarginChart.vue'
import ServiceChart from './components/ServiceChart.vue'

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

const data = metricsData as MonthData[]

// Month picker — year-level + individual months
const years = [...new Set(data.map(d => d.month.split('-')[0]))]

const monthOptions = computed(() => {
  const opts: { title: string; value: string }[] = []
  years.forEach(year => {
    opts.push({ title: `${year}: All Months`, value: `year-${year}` })
    data
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

// Filtered data for charts
const filteredData = computed(() => {
  if (isYearSelection.value) {
    return data.filter(d => d.month.startsWith(selectedYear.value!))
  }
  // Individual month: show 6 months preceding + current month
  const idx = data.findIndex(d => d.month === selectedMonth.value)
  const start = Math.max(0, idx - 6)
  return data.slice(start, idx + 1)
})

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

// Current data (for cards)
const currentData = computed(() => {
  if (isYearSelection.value) {
    const yearData = data.filter(d => d.month.startsWith(selectedYear.value!))
    return avg(yearData)
  }
  return data.find(d => d.month === selectedMonth.value)!
})

// Previous data (for delta arrows)
const previousData = computed((): MonthData | null => {
  if (isYearSelection.value) {
    // Compare to previous year
    const prevYear = String(+selectedYear.value! - 1)
    const prevYearData = data.filter(d => d.month.startsWith(prevYear))
    return prevYearData.length ? avg(prevYearData) : null
  }
  const idx = data.findIndex(d => d.month === selectedMonth.value)
  return idx > 0 ? data[idx - 1] : null
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
  return [
    { title: 'Gross Margin', value: c.grossMargin + '%', target: '15–18%', delta: delta(c.grossMargin, p?.grossMargin) },
    { title: 'Net Margin', value: c.netMargin + '%', target: '>12%', delta: delta(c.netMargin, p?.netMargin) },
    { title: 'Accessorial Recovery', value: c.accessorialRecovery + '%', target: '>85%', delta: delta(c.accessorialRecovery, p?.accessorialRecovery) },
    { title: 'Buy-Rate Variance', value: (c.buyRateVariance > 0 ? '+' : '') + c.buyRateVariance + '%', target: '±5%', delta: delta(c.buyRateVariance, p?.buyRateVariance) },
    { title: 'Revenue Quality (RPM)', value: '$' + c.revenueQualityRPM.toFixed(2), target: '>$2.50', delta: delta(c.revenueQualityRPM, p?.revenueQualityRPM) },
  ]
})

const serviceCards = computed(() => {
  const c = currentData.value
  const p = previousData.value
  return [
    { title: 'On-Time Delivery (OTD)', value: c.onTimeDelivery + '%', target: '>95%', delta: delta(c.onTimeDelivery, p?.onTimeDelivery) },
    { title: 'Exception Rate', value: c.exceptionRate + '%', target: '<3%', delta: deltaReverse(c.exceptionRate, p?.exceptionRate) },
    { title: 'First-Time-Right', value: c.firstTimeRight + '%', target: '>90%', delta: delta(c.firstTimeRight, p?.firstTimeRight) },
    { title: 'Dwell Time (Origin/Dest)', value: c.dwellTime + ' hrs', target: '<2 hrs', delta: deltaReverse(c.dwellTime, p?.dwellTime) },
  ]
})

// Performance Snapshot insights
const insights = computed(() => {
  // Find worst months for each category
  const worstMargin = [...data].sort((a, b) => a.grossMargin - b.grossMargin)[0]
  const worstOTD = [...data].sort((a, b) => a.onTimeDelivery - b.onTimeDelivery)[0]
  const worstException = [...data].sort((a, b) => b.exceptionRate - a.exceptionRate)[0]

  const fmtMonth = (m: string) => {
    const [y, mo] = m.split('-')
    return new Date(+y, +mo - 1).toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
  }

  return [
    {
      label: 'Top Margin Killer',
      icon: 'mdi-alert-circle',
      color: '#D32F2F',
      text: `Gross margin hit a low of ${worstMargin.grossMargin}% in ${fmtMonth(worstMargin.month)} while accessorial recovery dropped to ${worstMargin.accessorialRecovery}%, indicating unrecovered fees are compressing margins.`,
    },
    {
      label: 'Service Red Flag',
      icon: 'mdi-flag',
      color: '#E65100',
      text: `OTD dropped to ${worstOTD.onTimeDelivery}% in ${fmtMonth(worstOTD.month)} due to carrier capacity shifts, forcing expensive backup capacity purchases and hurting margins.`,
    },
    {
      label: 'Efficiency Gaps',
      icon: 'mdi-cog-outline',
      color: '#1565C0',
      text: `Exception rate spiked to ${worstException.exceptionRate}% in ${fmtMonth(worstException.month)} with first-time-right at just ${worstException.firstTimeRight}%, signaling process breakdowns that increase handling costs and rework.`,
    },
  ]
})
</script>

<template>
  <v-app>
    <v-app-bar color="white" elevation="1" density="comfortable">
      <v-app-bar-title class="font-weight-bold text-grey-darken-3">
        <v-icon class="mr-2" color="primary">mdi-truck-fast-outline</v-icon>
        Fast Forward Logistics
      </v-app-bar-title>
      <template #append>
        <v-select
          v-model="selectedMonth"
          :items="monthOptions"
          item-title="title"
          item-value="value"
          variant="outlined"
          density="compact"
          hide-details
          style="min-width: 220px"
          label="Period"
        />
      </template>
    </v-app-bar>

    <v-main>
      <v-container fluid class="pa-6">
        <!-- Financial Performance -->
        <h2 class="text-h6 font-weight-bold text-grey-darken-2 mb-3">Financial Performance</h2>
        <v-row>
          <v-col
            v-for="card in financialCards"
            :key="card.title"
            cols="12"
            sm="6"
            md="4"
            lg
          >
            <v-card variant="outlined" class="pa-4" rounded="lg">
              <div class="text-caption text-grey-darken-1 text-uppercase font-weight-medium">{{ card.title }}</div>
              <div class="d-flex align-center mt-1">
                <span class="text-h5 font-weight-bold">{{ card.value }}</span>
                <v-icon
                  v-if="card.delta"
                  :color="card.delta.color"
                  size="20"
                  class="ml-2"
                >{{ card.delta.icon }}</v-icon>
              </div>
              <div class="text-caption text-grey mt-1">Target: {{ card.target }}</div>
            </v-card>
          </v-col>
        </v-row>

        <!-- Service Reliability -->
        <h2 class="text-h6 font-weight-bold text-grey-darken-2 mt-8 mb-3">Service Reliability</h2>
        <v-row>
          <v-col
            v-for="card in serviceCards"
            :key="card.title"
            cols="12"
            sm="6"
            md="3"
          >
            <v-card variant="outlined" class="pa-4" rounded="lg">
              <div class="text-caption text-grey-darken-1 text-uppercase font-weight-medium">{{ card.title }}</div>
              <div class="d-flex align-center mt-1">
                <span class="text-h5 font-weight-bold">{{ card.value }}</span>
                <v-icon
                  v-if="card.delta"
                  :color="card.delta.color"
                  size="20"
                  class="ml-2"
                >{{ card.delta.icon }}</v-icon>
              </div>
              <div class="text-caption text-grey mt-1">Target: {{ card.target }}</div>
            </v-card>
          </v-col>
        </v-row>

        <!-- Charts -->
        <v-row class="mt-8">
          <v-col cols="12" md="6">
            <v-card variant="outlined" class="pa-4" rounded="lg">
              <div class="text-subtitle-1 font-weight-bold text-grey-darken-2 mb-3">Gross Margin Over Time</div>
              <GrossMarginChart :data="filteredData" :highlighted-month="selectedMonth" />
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card variant="outlined" class="pa-4" rounded="lg">
              <div class="text-subtitle-1 font-weight-bold text-grey-darken-2 mb-3">On-Time Delivery & Dwell Time</div>
              <ServiceChart :data="filteredData" :highlighted-month="selectedMonth" />
            </v-card>
          </v-col>
        </v-row>

        <!-- Performance Snapshot -->
        <h2 class="text-h6 font-weight-bold text-grey-darken-2 mt-8 mb-3">Performance Snapshot</h2>
        <v-card variant="outlined" rounded="lg">
          <v-list lines="three">
            <v-list-item
              v-for="insight in insights"
              :key="insight.label"
            >
              <template #prepend>
                <v-icon :color="insight.color" class="mr-3">{{ insight.icon }}</v-icon>
              </template>
              <v-list-item-title class="font-weight-bold">{{ insight.label }}</v-list-item-title>
              <v-list-item-subtitle class="text-wrap">{{ insight.text }}</v-list-item-subtitle>
            </v-list-item>
          </v-list>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<style>
html, body {
  background: #FAFAFA;
}
</style>
