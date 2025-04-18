<template>
  <div
    class="chart-container lg:flex-row md:flex-col items-start gap-[3.5rem] bg-white p-6 rounded-[0.75rem] shadow-md"
  >
    <!-- 차트 -->
    <div class="w-full md:w-full lg:w-[32rem] h-[32rem]">
      <canvas ref="chartRef" class="w-full h-full"></canvas>
    </div>

    <!-- 범례 -->
    <div
      class="w-full md:w-full lg:w-[14rem] space-y-[1rem] p-[1.5rem] rounded-[0.5rem] bg-[#f9fafb] shadow-md border border-[#d1d5db]"
    >
      <div
        v-for="(item, index) in chartData"
        :key="item.category"
        class="flex items-center justify-between gap-[1.5rem]"
      >
        <div class="flex items-center gap-[12px]">
          <span
            class="w-[16px] h-[16px] rounded-full inline-block"
            :style="{
              backgroundColor:
                colors[index % colors.length],
            }"
          ></span>
          <span
            class="text-sm font-medium text-gray-800 whitespace-nowrap"
          >
            {{ item.category }}
          </span>
        </div>
        <span
          class="text-sm font-semibold text-gray-900 whitespace-nowrap"
        >
          {{ formatPrice(item.amount) }} 원
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import {
  ref,
  watch,
  onMounted,
  onBeforeUnmount,
  nextTick,
} from 'vue';
import {
  Chart,
  ArcElement,
  Tooltip,
  Legend,
  DoughnutController,
} from 'chart.js';
import ChartDataLabels from 'chartjs-plugin-datalabels';

Chart.register(
  ArcElement,
  Tooltip,
  Legend,
  DoughnutController,
  ChartDataLabels
);

const props = defineProps({
  chartData: Array,
  colors: {
    type: Array,
    default: () => [
      '#a855f7',
      '#ec4899',
      '#f87171',
      '#f97316',
      '#eab308',
    ],
  },
});

const chartRef = ref(null);
let chartInstance = null;

const formatPrice = (val) => val.toLocaleString('ko-KR');

const drawChart = () => {
  const ctx = chartRef.value?.getContext('2d');
  if (!ctx) return;

  if (chartInstance) chartInstance.destroy();

  chartInstance = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: props.chartData.map((d) => d.category),
      datasets: [
        {
          data: props.chartData.map((d) => d.amount),
          backgroundColor: props.colors,
          borderWidth: 0,
        },
      ],
    },
    options: {
      cutout: '60%',
      plugins: {
        legend: { display: false },
        datalabels: {
          color: '#fff',
          font: { weight: 'bold', size: 12 },
          formatter: (value, context) => {
            const total = context.dataset.data.reduce(
              (a, b) => a + b,
              0
            );
            const percentage = (
              (value / total) *
              100
            ).toFixed(1);
            const label =
              context.chart.data.labels[context.dataIndex];
            return `${label}\n${percentage}%`;
          },
        },
      },
    },
    plugins: [ChartDataLabels],
  });
};

watch(
  () => [...props.chartData], // 얕은 복사해서 배열 값 변화 감지
  async () => {
    if (!props.chartData.length) return; // 데이터 없으면 차트 그리지 않음
    await nextTick();
    drawChart();
  },
  { immediate: true }
);

onBeforeUnmount(() => {
  if (chartInstance) chartInstance.destroy();
});
</script>

<style scoped>
.chart-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

/* ✅ 반응형: 화면 작아지면 세로 배치로 */
@media (max-width: 1200px) {
  .chart-container {
    flex-direction: column;
    align-items: center;
  }
}
</style>
