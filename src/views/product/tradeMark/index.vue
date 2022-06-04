<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" @click="showDialog"
      >添加</el-button
    >
    <el-table :data="list" style="width: 100%">
      <el-table-column type="index" label="序号" width="80px" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width">
      </el-table-column>
      <el-table-column prop="logoUrl" label="品牌LOGO" width="width">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ row }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTrademark(row)"
            >修改</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteTradeMark(row)">删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页器 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="getPageList"
      :current-page="pageNum"
      :page-sizes="[3, 5, 10]"
      :page-size="pageSize"
      :total="totalCount"
      layout="prev, pager, next, jumper, ->, sizes, total"
    >
    </el-pagination>
    <!-- 对话框 -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="tmForm" style="width: 80%" :rules="rules" ref="ruleForm">
        <el-form-item
          label="品牌名称"
          :label-width="formLabelWidth"
          prop="tmName"
        >
          <el-input v-model="tmForm.tmName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item
          label="品牌logo"
          :label-width="formLabelWidth"
          prop="logoUrl"
        >
          <!-- action 图片上传地址 -->
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "tradeMark",
  data() {
    var validateTmName = (rule, value, callback) => {
      if (value.length < 2 || value.length > 8) {
        callback(new Error('品牌名称2-8位'));
      } else {
        callback();
      }
    };
    return {
      totalCount: 0, // 总数量
      pageSize: 3, // 每页数量
      pageNum: 1, // 当前页码
      list: [], // 商品列表
      dialogFormVisible: false, // 对话框显示与隐藏
      formLabelWidth: "120px", // 输入框宽度
      // 收集品牌信息
      tmForm: {
        tmName: "",
        logoUrl: "", // 上传图片
      },
      // 表单验证
      rules: {
        //require:必须要校验字段（前面五角星有关系）  message 提示信息    trigger:用户行为设置（事件的设置:blur、change）
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          //自定义校验规则
          { validator: validateTmName, trigger: "change" },
        ],
        logoUrl: [
          { required: true, message: "请选择品牌logo", trigger: "change" },
        ],
      },
    };
  },
  methods: {
    // 获取品牌列表
    async getPageList(page = 1) {
      // 修改页码
      this.pageNum = page;
      // 解构参数
      const { pageNum, pageSize } = this;
      let result = await this.$API.trademark.reqTradeMark(pageNum, pageSize);
      this.totalCount = result.data.total;
      this.list = result.data.records;
    },

    // 切换每页展示条数
    handleSizeChange(limit) {
      this.pageSize = limit;
      this.getPageList();
    },

    // 显示弹出框
    showDialog() {
      this.dialogFormVisible = true;
      this.tmForm = { tmName: "", logoUrl: "" };
    },

    // 修改某一个品牌
    updateTrademark(row) {
      this.dialogFormVisible = true;
      // 将服务器返回的信息，直接赋值给了tmForm展示，需要浅拷贝
      this.tmForm = { ...row };
    },

    //图片上传成功
    handleAvatarSuccess(res, file) {
      // 收集图片数据，因为需要带给服务器
      this.tmForm.logoUrl = res.data;
    },
    // 上传图片之前
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },

    // 确定提交
    async addOrUpdate() {
      // 当全部验证通过后，才可以进行下一步操作
      this.$refs.ruleForm.validate(async (success) => {
        if (success) {
          this.dialogFormVisible = false;
          await this.$API.trademark.reqAddOrUpdate(this.tmForm);
          this.$message.success(
            this.tmForm.id ? "修改品牌成功" : "添加品牌成功"
          );
          //如果添加品牌： 停留在第一页，修改品牌应该留在当前页面
          this.getPageList(this.tmForm.id ? this.pageNum : 1);
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },

    // 删除
    deleteTradeMark(row) {
      this.$confirm(`你确定删除${row.tmName}？`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
          await this.$API.trademark.reqDeleteTradeMark(row.id);
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
          // 展示数量减一，并且停留在当前页面
          this.getPageList(this.list.length > 1 ? this.pageNum : this.pageNum-1);
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
    }
  },
  mounted() {
    this.getPageList();
  },
};
</script>

<style>
.el-pagination {
  margin-top: 20px;
  text-align: center;
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>