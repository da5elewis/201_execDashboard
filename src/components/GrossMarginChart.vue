<script setup lang="ts">
import { computed } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler,
} from 'chart.js'

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend, Filler)

interface MonthData {
  month: string
  grossMargin: number
}

const props = defineProps<{
  data: MonthData[]
  nationalData?: MonthData[]
  highlightedMonth?: string
}>()

const emit = defineEmits<{
  (e: 'select-month', month: string): void
}>()

function onChartClick(_event: any, elements: any[]) {
  if (elements.length > 0) {
    const idx = elements[0].index
    if (idx >= 0 && idx < props.data.length) {
      emit('select-month', props.data[idx].month)
    }
  }
}

const chartData = computed(() => {
  const highlightIdx = props.highlightedMonth && props.highlightedMonth !== 'all'
    ? props.data.findIndex(d => d.month === props.highlightedMonth)
    : -1

  return {
    labels: props.data.map(d => {
      const [y, m] = d.month.split('-')
      return new Date(+y, +m - 1).toLocaleDateString('en-US', { month: 'short', year: '2-digit' })
    }),
    datasets: [
      {
        label: 'Gross Margin %',
        data: props.data.map(d => d.grossMargin),
        borderColor: '#2E7D32',
        backgroundColor: 'rgba(46,125,50,0.08)',
        fill: true,
        tension: 0.3,
        pointRadius: props.data.map((_, i) => i === highlightIdx ? 8 : 3),
        pointBackgroundColor: props.data.map((_, i) => i === highlightIdx ? '#1B5E20' : '#2E7D32'),
        pointBorderColor: props.data.map((_, i) => i === highlightIdx ? '#FDD835' : '#2E7D32'),
        pointBorderWidth: props.data.map((_, i) => i === highlightIdx ? 3 : 0),
      },
      ...(props.nationalData ? [{
        label: 'National Gross Margin %',
        data: props.nationalData.map(d => d.grossMargin),
        borderColor: 'rgba(46,125,50,0.3)',
        backgroundColor: 'transparent',
        fill: false,
        tension: 0.3,
        pointRadius: 0,
      }] : []),
      {
        label: 'Target Min (15%)',
        data: props.data.map(() => 15),
        borderColor: '#BDBDBD',
        borderDash: [6, 4],
        pointRadius: 0,
        fill: false,
      },
      {
        label: 'Target Max (18%)',
        data: props.data.map(() => 18),
        borderColor: '#BDBDBD',
        borderDash: [6, 4],
        pointRadius: 0,
        fill: false,
      },
    ],
  }
})

const hasNational = computed(() => !!props.nationalData)

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  onClick: onChartClick,
  plugins: {
    legend: { display: false },
    title: { display: false },
  },
  scales: {
    y: {
      min: 12,
      max: 20,
      ticks: { callback: (v: unknown) => v + '%' },
      grid: { color: '#F5F5F5' },
    },
    x: {
      grid: { display: false },
    },
  },
}
</script>

<template>
  <div>
    <div style="height: 300px">
      <Line :data="chartData" :options="chartOptions" />
    </div>
    <div class="chart-legend">
      <div class="legend-col">
        <div class="legend-item"><span class="legend-swatch" style="background: #2E7D32"></span> Gross Margin %</div>
        <div v-if="hasNational" class="legend-item"><span class="legend-swatch" style="background: rgba(46,125,50,0.3)"></span> National Gross Margin %</div>
      </div>
      <div class="legend-col">
        <div class="legend-item"><span class="legend-swatch legend-dashed" style="border-color: #BDBDBD"></span> Target Max (18%)</div>
        <div class="legend-item"><span class="legend-swatch legend-dashed" style="border-color: #BDBDBD"></span> Target Min (15%)</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chart-legend {
  display: flex;
  justify-content: space-between;
  padding: 8px 12px 0;
  font-size: 12px;
  color: #616161;
}
.legend-col {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
}
.legend-swatch {
  display: inline-block;
  width: 24px;
  height: 3px;
  border-radius: 2px;
}
.legend-dashed {
  background: transparent;
  border-top: 2px dashed;
  height: 0;
}
</style>
