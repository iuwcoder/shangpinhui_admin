<template>
  <div>
    <el-table :data="records" style="width: 100%" border>
      <el-table-column
        type="index"
        label="序号"
        width="80"
        align="center"
      ></el-table-column>
      <el-table-column prop="skuName" label="名称" width="width">
      </el-table-column>
      <el-table-column prop="skuDesc" label="描述" width="width">
      </el-table-column>
      <el-table-column prop="prop" label="默认图片" width="125">
        <template slot-scope="{ row }">
          <img
            :src="row.skuDefaultImg"
            alt=""
            style="width: 100px; height: 100px"
          />
        </template>
      </el-table-column>
      <el-table-column prop="weight" label="重量" width="80"> </el-table-column>
      <el-table-column prop="price" label="价格" width="80"> </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ row }">
          <el-button
            type="success"
            icon="el-icon-bottom"
            size="mini"
            v-if="row.isSale == 0"
            @click="sale(row)"
          ></el-button>
          <el-button
            type="success"
            icon="el-icon-top"
            size="mini"
            v-else
            @click="cancel(row)"
          ></el-button>
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="edit"
          ></el-button>
          <el-button
            type="info"
            icon="el-icon-info"
            size="mini"
            title="查看详情"
            @click="getSkuInfo(row)"
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            title="删除"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 右侧抽屉 -->
    <el-drawer
      :visible.sync="drawer"
      
      :show-close="false"
      :before-close="handleClose"
      size="40%"
    >
      <el-row>
        <el-col :span="5">名称</el-col>
        <el-col :span="16">{{ skuInfo.skuName }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">描述</el-col>
        <el-col :span="16">{{ skuInfo.skuDesc }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">价格</el-col>
        <el-col :span="16">{{ skuInfo.price }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">平台属性</el-col>
        <el-col :span="16">
          <template>
            <el-tag
              type="success"
              v-for="attr in skuInfo.skuAttrValueList"
              :key="attr.id"
              style="margin:0 7px 10px 0"
              >{{ attr.attrId }}-{{ attr.valueId }}</el-tag
            >
          </template>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="5">商品图片</el-col>
        <el-col :span="16">
          <el-carousel height="450px">
            <el-carousel-item
              v-for="item in skuInfo.skuImageList"
              :key="item.id">
                <img :src="item.imgUrl" alt="" style="width:100%; height:450px"/>
            </el-carousel-item>
          </el-carousel>
        </el-col>
      </el-row>
    </el-drawer>
    <!-- 分页器 -->
    <el-pagination
      style="text-align: center"
      @size-change="handleSizeChange"
      @current-change="getSkuList"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      :total="total"
      layout="prev, pager, next, jumper, ->, sizes, total"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: "Sku",
  data() {
    return {
      page: 1,
      limit: 5,
      total: 0,
      records: [], // sku列表数据
      skuInfo: {}, // sku商品详情
      drawer: false, // 控制抽屉显示
    };
  },
  methods: {
    // 获取数据
    async getSkuList(pages = 1) {
      this.page = pages;
      let result = await this.$API.sku.reqSkuList(this.page, this.limit);
      this.total = result.data.total;
      this.records = result.data.records;
    },
    // 修改展示条数
    handleSizeChange(limit) {
      this.limit = limit;
      this.getSkuList();
    },

    // 上架
    async sale(row) {
      await this.$API.sku.reqSale(row.id);
      row.isSale = 1;
      this.$message.success("上架成功");
    },
    // 下架
    async cancel(row) {
      await this.$API.sku.reqCancel(row.id);
      row.isSale = 0;
      this.$message.success("下架成功");
    },
    edit() {
      this.$message("正在开发中");
    },

    // 获取sku详情
    async getSkuInfo(row) {
      this.drawer = true;
      let result = await this.$API.sku.reqSkuById(row.id);
      this.skuInfo = result.data;
    },

    handleClose(done) {
      this.skuInfo = ''
      done()
    }
  },
  mounted() {
    this.getSkuList();
  },
};
</script>

<style lang="scss">
.el-carousel__item img {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
</style>

<style scoped>
.el-row .el-col-5 {
  font-size: 18px;
  text-align: right;
}
.el-col {
  margin: 10px;
}

/* >>> .el-carousel__button {
  width: 10px;
  height: 10px;
  background: red;
  border-radius: 50%;
} */
</style>