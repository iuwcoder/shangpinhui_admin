<template>
  <div class="echarts" ref="charts"></div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "lineCharts",
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
          type: "line",
          data: [15, 7, 20, 13, 38, 10, 25, 15, 44],
          smooth: true,
          // 拐点的样式
          itemStyle: {
            opacity: 0,
          },
          // 线条的样式
          lineStyle: {
            color: "rgb(170, 116, 231)",
          },
          // 填充颜色
          areaStyle: {
            color: {
              type: "linear",
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              colorStops: [
                {
                  offset: 0,
                  color: "rgb(170, 116, 231)", // 0% 处的颜色
                },
                {
                  offset: 1,
                  color: "#fff", // 100% 处的颜色
                },
              ],
              global: false, // 缺省为 false
            },
          },
        },
      ],
      // 布局调试
      gird: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
      },
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