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
  onTimeDelivery: number
  dwellTime: number
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
        label: 'On-Time Delivery %',
        data: props.data.map(d => d.onTimeDelivery),
        borderColor: '#1565C0',
        backgroundColor: 'rgba(21,101,192,0.08)',
        fill: true,
        tension: 0.3,
        pointRadius: props.data.map((_, i) => i === highlightIdx ? 8 : 3),
        pointBackgroundColor: props.data.map((_, i) => i === highlightIdx ? '#0D47A1' : '#1565C0'),
        pointBorderColor: props.data.map((_, i) => i === highlightIdx ? '#FDD835' : '#1565C0'),
        pointBorderWidth: props.data.map((_, i) => i === highlightIdx ? 3 : 0),
        yAxisID: 'y',
      },
      {
        label: 'Dwell Time (hrs)',
        data: props.data.map(d => d.dwellTime),
        borderColor: '#FF8F00',
        backgroundColor: 'rgba(255,143,0,0.08)',
        fill: true,
        tension: 0.3,
        pointRadius: props.data.map((_, i) => i === highlightIdx ? 8 : 3),
        pointBackgroundColor: props.data.map((_, i) => i === highlightIdx ? '#E65100' : '#FF8F00'),
        pointBorderColor: props.data.map((_, i) => i === highlightIdx ? '#FDD835' : '#FF8F00'),
        pointBorderWidth: props.data.map((_, i) => i === highlightIdx ? 3 : 0),
        yAxisID: 'y1',
      },
      ...(props.nationalData ? [
        {
          label: 'National OTD %',
          data: props.nationalData.map(d => d.onTimeDelivery),
          borderColor: 'rgba(21,101,192,0.3)',
          backgroundColor: 'transparent',
          fill: false,
          tension: 0.3,
          pointRadius: 0,
          yAxisID: 'y',
        },
        {
          label: 'National Dwell (hrs)',
          data: props.nationalData.map(d => d.dwellTime),
          borderColor: 'rgba(255,143,0,0.3)',
          backgroundColor: 'transparent',
          fill: false,
          tension: 0.3,
          pointRadius: 0,
          yAxisID: 'y1',
        },
      ] : []),
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
      type: 'linear' as const,
      position: 'left' as const,
      min: 80,
      max: 100,
      ticks: { callback: (v: unknown) => v + '%' },
      grid: { color: '#F5F5F5' },
      title: { display: true, text: 'OTD %' },
    },
    y1: {
      type: 'linear' as const,
      position: 'right' as const,
      min: 0,
      max: 4,
      ticks: { callback: (v: unknown) => v + 'h' },
      grid: { drawOnChartArea: false },
      title: { display: true, text: 'Dwell (hrs)' },
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
        <div class="legend-item"><span class="legend-swatch" style="background: #1565C0"></span> On-Time Delivery %</div>
        <div v-if="hasNational" class="legend-item"><span class="legend-swatch" style="background: rgba(21,101,192,0.3)"></span> National OTD %</div>
      </div>
      <div class="legend-col">
        <div class="legend-item"><span class="legend-swatch" style="background: #FF8F00"></span> Dwell Time (hrs)</div>
        <div v-if="hasNational" class="legend-item"><span class="legend-swatch" style="background: rgba(255,143,0,0.3)"></span> National Dwell (hrs)</div>
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
</style>
