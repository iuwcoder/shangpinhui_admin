<template>
  <div>
    <el-row :gutter="10">
      <el-col :span="6">
        <el-card>
          <detail title="总销售额" count="￥126550">
            <template slot="charts">
              <span>周同比 56.67</span>
              <i class="el-icon-top"></i>
              <span>日同比 19.15</span>
              <i class="el-icon-bottom"></i>
            </template>
            <template slot="footer">
              <span>日销售额 ￥11567</span>
            </template>
          </detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <detail title="访问量" count="￥88460">
            <template slot="charts">
              <lineCharts></lineCharts>
            </template>
            <template slot="footer">
              <span>日访问量 567</span>
            </template>
          </detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <detail title="支付笔数" count="￥18460">
            <template slot="charts">
              <barCharts></barCharts>
            </template>
            <template slot="footer">
              <span>转化率 60%</span>
            </template>
          </detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <detail title="运营活动效果" count="78%">
            <template slot="charts">
              <progressCharts></progressCharts>
            </template>
            <template slot="footer">
              <span>周同比 56.67%</span>
              <i class="el-icon-top"></i>
              <span>日同比 19.15%</span>
              <i class="el-icon-bottom"></i>
            </template>
          </detail>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Detail from "./Detail";
import lineCharts from "./lineCharts";
import barCharts from "./barCharts";
import progressCharts from './progressCharts'

export default {
  name: "Card",
  components: {
    Detail,
    lineCharts,
    barCharts,
    progressCharts
  },
};
</script>

<style scoped>
.el-icon-top {
  color: red;
  padding-right: 10px;
}
.el-icon-bottom {
  color: rgb(72, 204, 10);
}

</style>