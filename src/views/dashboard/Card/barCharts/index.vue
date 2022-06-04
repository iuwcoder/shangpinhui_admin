<template>
  <div class="echarts" ref="charts"></div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "barCharts",
  mounted() {
    // 初始化echarts实例
    let lineCharts = echarts.init(this.$refs.charts);
    lineCharts.setOption({
      xAxis: {
        show: false,
        type: "category",
        boundaryGap: false
      },
      yAxis: {
        show: false,
      },
      series: [
        {
          type: "bar",
          data: [15, 7, 20, 13, 38, 10, 25, 15, 44],
        },
      ],
      // 布局调试
      gird: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
      },
      tooltip: {}
    });
  },
};
</script>

<style scoped>
.echarts {
  width: 100%;
  height: 100%;
}
</style>