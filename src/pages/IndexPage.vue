<template>
  <q-page class="q-pa-md">
    <div>
      <h5>CSV Upload & Chart Viewer</h5>

      <div>
        <q-file
          filled
          v-model="csvFile"
          label="Upload CSV File"
          accept=".csv"
          @update:model-value="handleFileUpload"
        />
      </div>

      <div v-if="headers.length > 1" class="q-mt-md">
        <div>
          <q-select
            v-model="xColumn"
            :options="headers"
            label="X-Axis"
            filled
          />
        </div>

        <div class="q-mt-sm">
          <q-select
            v-model="yColumn"
            :options="headers"
            label="Y-Axis"
            filled
          />
        </div>

        <div class="q-mt-md">
          <q-btn
            label="Line Chart"
            color="primary"
            @click="drawChart('line')"
            :disable="!xColumn || !yColumn"
          />
          <q-btn
            label="Bar Chart"
            color="secondary"
            @click="drawChart('bar')"
            :disable="!xColumn || !yColumn"
            class="q-ml-sm"
          />
        </div>
      </div>

      <div v-show="dataRows.length && xColumn && yColumn" class="q-mt-md">
        <canvas ref="chartCanvas" style="max-height: 400px; background: #fff; border: 1px solid #ccc;" />
        <div class="q-mt-md">
          <q-btn
            label="Export as PNG"
            color="green"
            @click="exportChart"
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, nextTick } from 'vue';
import Papa from 'papaparse';
import { Chart } from 'chart.js/auto';

const csvFile = ref(null);
const dataRows = ref([]);
const headers = ref([]);
const xColumn = ref(null);
const yColumn = ref(null);
const chart = ref(null);
const chartCanvas = ref(null);

function handleFileUpload(file) {
  if (!file) return;

  const reader = new FileReader();
  reader.onload = () => {
    Papa.parse(reader.result, {
      header: true,
      skipEmptyLines: true,
      complete(results) {
        if (results.data.length > 0) {
          dataRows.value = results.data;
          headers.value = Object.keys(results.data[0]);
          xColumn.value = null;
          yColumn.value = null;
        }
      }
    });
  };
  reader.readAsText(file);
}

async function drawChart(type) {
  if (!xColumn.value || !yColumn.value) return;

  if (chart.value) {
    chart.value.destroy();
  }

  await nextTick();

  const ctx = chartCanvas.value.getContext('2d');
  const xData = dataRows.value.map(row => row[xColumn.value]);
  const yData = dataRows.value.map(row => parseFloat(row[yColumn.value]));

  chart.value = new Chart(ctx, {
    type,
    data: {
      labels: xData,
      datasets: [{
        label: `${yColumn.value} vs ${xColumn.value}`,
        data: yData,
        backgroundColor: 'rgba(63, 81, 181, 0.5)',
        borderColor: 'rgba(63, 81, 181, 1)',
        borderWidth: 2,
        fill: false,
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'top'
        }
      },
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });
}

function exportChart() {
  if (!chart.value) return;

  const a = document.createElement('a');
  a.href = chart.value.toBase64Image();
  a.download = 'chart.png';
  a.click();
}
</script>

<style scoped>
canvas {
  width: 100% !important;
  height: auto !important;
  margin-top: 10px;
}
</style>
