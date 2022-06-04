<template>
  <div>
    <div class="card-header">
      <span>{{title}}</span>
      <i class="el-icon-warning-outline"></i>
    </div>
    <div class="card-content">{{count}}</div>
    <div class="card-charts">
      <slot name="charts"></slot>
    </div>
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "Detail",
  props: ['title', 'count']

}
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  color: rgb(167, 167, 167);
  font-size: 16px;
}
.card-content {
  font-size: 30px;
  margin: 10px 0;
}
.card-charts {
  margin: 10px 0;
  height: 60px;
}
.card-footer {
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  padding-top: 10px;
}

</style>