<script setup lang="ts">
interface RegionRow {
  region: string
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

defineProps<{
  rows: RegionRow[]
}>()

const columns = [
  { key: 'grossMargin', label: 'Gross Margin', format: (v: number) => v + '%', good: (v: number) => v >= 15 },
  { key: 'netMargin', label: 'Net Margin', format: (v: number) => v + '%', good: (v: number) => v > 12 },
  { key: 'accessorialRecovery', label: 'Accessorial Recovery', format: (v: number) => v + '%', good: (v: number) => v > 85 },
  { key: 'buyRateVariance', label: 'Buy-Rate Var.', format: (v: number) => (v > 0 ? '+' : '') + v + '%', good: (v: number) => Math.abs(v) <= 5 },
  { key: 'revenueQualityRPM', label: 'RPM', format: (v: number) => '$' + v.toFixed(2), good: (v: number) => v > 2.5 },
  { key: 'onTimeDelivery', label: 'OTD', format: (v: number) => v + '%', good: (v: number) => v > 95 },
  { key: 'exceptionRate', label: 'Exception Rate', format: (v: number) => v + '%', good: (v: number) => v < 3 },
  { key: 'firstTimeRight', label: 'First-Time-Right', format: (v: number) => v + '%', good: (v: number) => v > 90 },
  { key: 'dwellTime', label: 'Dwell Time', format: (v: number) => v + 'h', good: (v: number) => v < 2 },
]

function cellColor(col: typeof columns[number], value: number): string {
  return col.good(value) ? 'rgba(46,125,50,0.15)' : 'rgba(211,47,47,0.15)'
}

function getValue(row: RegionRow, key: string): number {
  return (row as any)[key]
}
</script>

<template>
  <div class="heatmap-wrapper">
    <table class="heatmap-table">
      <thead>
        <tr>
          <th class="region-header"></th>
          <th v-for="col in columns" :key="col.key" class="col-header">{{ col.label }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in rows" :key="row.region">
          <td class="region-label">{{ row.region }}</td>
          <td
            v-for="col in columns"
            :key="col.key"
            class="cell"
            :style="{ background: cellColor(col, getValue(row, col.key)) }"
          >
            {{ col.format(getValue(row, col.key)) }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.heatmap-wrapper {
  overflow-x: auto;
}
.heatmap-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 4px;
  font-size: 13px;
}
.region-header {
  width: 80px;
}
.col-header {
  text-align: center;
  padding: 6px 8px;
  font-weight: 600;
  color: #616161;
  font-size: 11px;
  white-space: nowrap;
}
.region-label {
  font-weight: 600;
  color: #424242;
  padding: 6px 10px;
  white-space: nowrap;
}
.cell {
  text-align: center;
  padding: 6px 8px;
  border-radius: 4px;
  font-weight: 500;
}
</style>
