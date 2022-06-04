<template>
  <div>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <!-- 头部左侧内容 -->
        <el-tabs class="tab" v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="销售额" name="sale"></el-tab-pane>
          <el-tab-pane label="访问量" name="visitor"></el-tab-pane>
        </el-tabs>
        <!-- 头部右侧内容 -->
        <div class="right">
          <span @click="setDay">今日</span>
          <span @click="setWeek">本周</span>
          <span @click="setMonth">本月</span>
          <span @click="setYear">本年</span>
          <el-date-picker
            class="date"
            type="datetimerange"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :default-time="['12:00:00']"
            v-model="data"
            size="mini"
            value-format="yyyy-MM-dd"
          >
          </el-date-picker>
        </div>
      </div>
      <div>
        <el-row :gutter="10">
          <el-col :span="18">
            <!-- 柱状图 -->
            <div class="charts" ref="charts"></div>
          </el-col>
          <el-col :span="6" class="right">
            <h3>门店{{ title }}排名</h3>
            <ul>
              <li>
                <span class="index">1</span>
                <span>肯德基</span>
                <span class="value">22345</span>
              </li>
              <li>
                <span class="index">2</span>
                <span>麦当劳</span>
                <span class="value">19545</span>
              </li>
              <li>
                <span class="index">3</span>
                <span>一点点</span>
                <span class="value">18975</span>
              </li>
              <li>
                <span class="index">4</span>
                <span>茶百道</span>
                <span class="value">17589</span>
              </li>
              <li>
                <span class="index">5</span>
                <span>奈雪的茶</span>
                <span class="value">9865</span>
              </li>
              <li>
                <span class="index">6</span>
                <span>CoCo都可</span>
                <span class="value">8578</span>
              </li>
              <li>
                <span class="index">7</span>
                <span>蜜雪冰城</span>
                <span class="value">6345</span>
              </li>
            </ul>
          </el-col>
        </el-row>
      </div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from "echarts";
import dayjs from "dayjs";

export default {
  name: "Sale",
  data() {
    return {
      activeName: "sale",
      mycharts: null,
      data: [], // 收集日历的数据
    };
  },
  methods: {
    // 获取今日时间
    setDay() {
      const day = dayjs().format("YYYY-MM-DD");
      this.data = [day, day];
    },
    // 获取本周
    setWeek() {
      const start = dayjs().day(1).format("YYYY-MM-DD");
      const end = dayjs().day(7).format("YYYY-MM-DD");
      this.data = [start, end];
    },
    // 本月
    setMonth() {
      const start = dayjs().startOf("month").format("YYYY-MM-DD");
      const end = dayjs().endOf("month").format("YYYY-MM-DD");
      this.data = [start, end];
    },
    // 本年
    setYear() {
      const start = dayjs().startOf("month").format("YYYY-MM-DD");
      const end = dayjs().endOf("month").format("YYYY-MM-DD");
      this.data = [start, end];
    },
  },
  mounted() {
    this.mycharts = echarts.init(this.$refs.charts);
    this.mycharts.setOption({
      title: {
        text: this.title + "趋势",
      },
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
      },
      grid: {
        left: "3%",
        right: "4%",
        bottom: "3%",
        containLabel: true,
      },
      xAxis: [
        {
          type: "category",
          data: [
            "一月",
            "二月",
            "三月",
            "四月",
            "五月",
            "六月",
            "七月",
            "八月",
            "九月",
            "十月",
            "十一月",
            "十二月",
          ],
          axisTick: {
            alignWithLabel: true,
          },
        },
      ],
      yAxis: [
        {
          type: "value",
        },
      ],
      series: [
        {
          name: "Direct",
          type: "bar",
          barWidth: "60%",
          data: [10, 52, 200, 304, 390, 330, 220, 95, 52, 200, 334, 158],
        },
      ],
    });
  },
  computed: {
    title() {
      return this.activeName == "sale" ? "销售额" : "访问量";
    },
  },
  watch: {
    title() {
      // 图标的值可以再次修改
      this.mycharts.setOption({
        title: {
          text: this.title + "趋势",
        },
      });
    },
  },
};
</script>

<style>
.clearfix {
  display: flex;
  justify-content: space-between;
  position: relative;
}
.tab {
  width: 100%;
}
.right {
  position: absolute;
  right: 0;
}
.date {
  width: 200px;
  margin: 0 20px;
}
.right span {
  margin: 0 10px;
}
.charts {
  width: 100%;
  height: 300px;
}

.text {
  font-size: 14px;
}
h3 {
  margin-top: 0;
}
ul {
  list-style: none;
  width: 100%;
  height: 250px;
  margin: 0;
  padding: 0;
}

ul li {
  height: 8%;
  margin: 15px 0;
}

.index {
  float: left;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background-color: #333;
  color: #fff;
  text-align: center;
  font-size: 14px;
  line-height: 20px;
}
.value {
  float: right;
}
/* .item {
  margin-bottom: 18px;
} */

/* .clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
} */

.box-card {
  /* width: 480px; */
  margin: 10px 0;
}
</style>