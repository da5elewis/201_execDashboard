<script setup lang="ts">
import { computed } from 'vue'
import { Bar } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  Title,
  Tooltip,
  Legend,
} from 'chart.js'

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)

interface MonthData {
  month: string
  onTimeDelivery: number
  dwellTime: number
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
        label: 'On-Time Delivery %',
        data: props.data.map(d => d.onTimeDelivery),
        backgroundColor: props.data.map((_, i) => i === highlightIdx ? '#0D47A1' : '#1565C0'),
        borderColor: props.data.map((_, i) => i === highlightIdx ? '#FDD835' : 'transparent'),
        borderWidth: props.data.map((_, i) => i === highlightIdx ? 3 : 0),
        yAxisID: 'y',
      },
      {
        label: 'Dwell Time (hrs)',
        data: props.data.map(d => d.dwellTime),
        backgroundColor: props.data.map((_, i) => i === highlightIdx ? '#E65100' : '#FF8F00'),
        borderColor: props.data.map((_, i) => i === highlightIdx ? '#FDD835' : 'transparent'),
        borderWidth: props.data.map((_, i) => i === highlightIdx ? 3 : 0),
        yAxisID: 'y1',
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
  <div style="height: 300px">
    <Bar :data="chartData" :options="chartOptions" />
  </div>
</template>
