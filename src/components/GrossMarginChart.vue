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
}>()

const chartData = computed(() => ({
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
      pointRadius: 3,
      pointBackgroundColor: '#2E7D32',
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
}))

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
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
