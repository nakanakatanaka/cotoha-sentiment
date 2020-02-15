<template>
  <div class="chart">
      <line-chart
        :chart-data="dataCollection"
      ></line-chart>
  </div>
</template>

<script>
import LineChart from './LineChart.js';

export default {
    components: {
        LineChart
    },
    props: {
        dataCollection: null
    }
}
</script>