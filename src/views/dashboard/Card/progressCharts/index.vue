<template>
  <div class="echarts" ref="charts"></div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "progressCharts",
  mounted() {
    // 初始化echarts实例
    let lineCharts = echarts.init(this.$refs.charts);
    lineCharts.setOption({
      xAxis: {
        show: false,
        // 最小值与最大值的设置
        min: 0,
        max: 100
      },
      yAxis: {
        show: false,
        // 均分
        type: "category",
      },
      series: [
        {
          type: "bar",
          data: [78],
          // 柱状图的宽度
          barWidth: 10,
          color: 'yellowgreen',
          // 背景颜色
          showBackground: {
            color: '#eee'
          },
          // 文本标签
          label: {
            show: true,
            // 发改变文本内容
            formatter: '|',
            position: 'right'
          }
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