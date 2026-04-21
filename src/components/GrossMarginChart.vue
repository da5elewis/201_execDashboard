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

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  onClick: onChartClick,
  plugins: {
    legend: { display: true, position: 'bottom' as const },
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
  <div style="height: 300px">
    <Line :data="chartData" :options="chartOptions" />
  </div>
</template>
