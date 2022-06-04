<template>
  <div>
    <el-card style="margin: 20px 0">
      <CategorySelect @getCategoryId="getCategoryId" :show="scene == 0" />
    </el-card>
    <el-card style="margin: 20px 0">
      <div v-show="scene == 0">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addSpu"
          >添加SPU</el-button
        >
        <el-table style="width: 100%; margin: 20px 0" border :data="records">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="spuName" label="spu名称" width="150">
          </el-table-column>
          <el-table-column prop="description" label="spu描述" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="300">
            <template slot-scope="{ row }">
              <el-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加"
                @click="addSku(row)"
              ></el-button>
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改"
                @click="updateSpu(row)"
              ></el-button>
              <el-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前spu全部sku列表"
                @click="handleDialog(row)"
              ></el-button>
              <el-popconfirm
                style="margin-left: 10px"
                title="确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页器 -->
        <el-pagination
          style="text-align: center"
          @size-change="handleSizeChange"
          @current-change="getSpuList"
          :current-page="page"
          :page-sizes="[3, 5, 10]"
          :page-size="limit"
          :total="total"
          layout="prev, pager, next, jumper, ->, sizes, total"
        >
        </el-pagination>
      </div>
      <SpuForm v-show="scene == 1" @changeScene="changeScene" ref="spu"></SpuForm>
      <SkuForm v-show="scene == 2" ref="sku" @changeScenes="changeScenes"></SkuForm>
      <!-- 展示SKU列表的对话框 -->
      <el-dialog :title="`${spu.spuName}的SKU列表`" :visible.sync="dialogTableVisible" :before-close="close">
        <el-table :data="skuList" style="width: 100%" border v-loading="loading">
          <el-table-column prop="skuName" label="名称" width="width"></el-table-column>
          <el-table-column prop="price" label="价格" width="150"></el-table-column>
          <el-table-column prop="weight" label="重量" width="150"></el-table-column>
          <el-table-column label="默认图片" width="width">
            <template slot-scope="{row}">
              <img :src="row.skuDefaultImg" alt="" style="width:100px;height:100px">
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";

export default {
  name: "Spu",
  components: {
    SpuForm,
    SkuForm,
  },
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      isShow: true, // 控制三级联动
      page: 1, // 默认页码
      limit: 3, // 每页数量
      records: [], // spu列表数据
      total: 0, // 总条数
      scene: 0, // 0显示SPU列表数据   1添加/修改SPU  2添加SKU
      dialogTableVisible: false, //控制对话框显示
      spu: {},
      skuList: [], //sku列表数据
      loading: true
    };
  },
  methods: {
    //三级联动 自定义事件的回调
    getCategoryId({ categoryId, level }) {
      //区分三级分类相应的id，以及父组件进行存储
      if (level == 1) {
        this.category1Id = categoryId;
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        //发请求获取spu的属性数据
        this.getSpuList();
      }
    },

    // 获取spu列表数据
    async getSpuList(pages = 1) {
      this.page = pages;
      const { page, limit, category3Id } = this;
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      this.total = result.data.total;
      this.records = result.data.records;
    },

    // 改变每页条数
    handleSizeChange(limit) {
      this.limit = limit;
      this.getSpuList();
    },

    // 添加SPU按钮的回调
    addSpu() {
      this.scene = 1;
      // 通知子组件SpuForm发请求
      this.$refs.spu.addSpuData(this.category3Id);
    },

    // 修改SPU
    updateSpu(row) {
      this.scene = 1;
      this.$refs.spu.initSpuData(row);
    },

    // SpuForm的自定义事件的回调
    changeScene(scene) {
      // 切换场景
      this.scene = scene;
      // 重新发送请求，请展示在当前页码
      this.getSpuList(this.page);
    },

    // 删除SPU
    async deleteSpu(row) {
      await this.$API.spu.reqDeleteSpu(row.id);
      this.$message.success("删除成功");
      this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
    },

    // 添加SKU
    addSku(row) {
      this.scene = 2;
      // 调用子组件方法
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },

    // SpuForm的自定义事件的回调
    changeScenes(scene) {
      // 切换场景
      this.scene = scene;
      // 重新发送请求，请展示在当前页码
      // this.getSpuList(this.page);
    },

    // 查看SKU对话框
    async handleDialog(row) {
      this.dialogTableVisible = true
      // 保存spu信息
      this.spu = row
      let result = await this.$API.spu.reqSkuList(row.id)
      this.skuList = result.data
      this.loading = false
    },
    // 关闭对话框的回调
    close(done) {
      this.loading = true
      this.skuList = ''
      done()
    }
  },
};
</script>

<style>
</style>