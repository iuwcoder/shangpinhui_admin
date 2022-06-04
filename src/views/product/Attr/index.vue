<template>
  <div>
    <el-card style="margin: 20px 0">
      <CategorySelect @getCategoryId="getCategoryId" :show="isShow"/>
    </el-card>
    <el-card>
      <div v-show="isShow">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addAttr"
          >添加属性</el-button
        >
        <el-table :data="attrList" style="width: 100%; margin: 20px 0" border>
          <el-table-column label="序号" width="80" type="index" align="center">
          </el-table-column>
          <el-table-column prop="attrName" label="属性名称" width="150">
          </el-table-column>
          <el-table-column prop="prop" label="属性值列表" width="width">
            <template slot-scope="{ row }">
              <el-tag
                type="success"
                v-for="attrValue in row.attrValueList"
                :key="attrValue.id"
                style="margin: 0 15px"
                >{{ attrValue.valueName }}</el-tag
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="150">
            <template slot-scope="{ row }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateAttr(row)"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 添加/修改属性 -->
      <div v-show="!isShow">
        <el-form :inline="true" ref="form" :model="attrInfo" label-width="80px">
          <el-form-item label="属性名">
            <el-input
              v-model="attrInfo.attrName"
              placeholder="请输入属性名"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button type="primary" icon="el-icon-plus" :disabled="!attrInfo.attrName" @click="addAttrValue">添加属性值</el-button>
        <el-button>取消</el-button>
        <el-table
          :data="attrInfo.attrValueList"
          style="width: 100%; margin: 20px 0"
          border
        >
          <el-table-column type="index" align="center" label="序号" width="80">
          </el-table-column>
          <el-table-column label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <el-input
                v-model="row.valueName"
                placeholder="请输入属性值名称"
                size="mini"
                v-if="row.flag"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
                :ref="$index"
              ></el-input>
              <span v-else @click="toEdit(row, $index)" style="display:block">{{row.valueName}}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" prop="prop" width="width">
            <template slot-scope="{ row, $index}">
              <el-popconfirm :title="`确定删除${row.valueName}？`" @onConfirm="deleteAttrValue($index)">
                <el-button slot="reference" type="danger" icon="el-icon-delete" size="mini"></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button type="primary" @click="addOrUpdate" :disabled="attrInfo.attrValueList.length<1">保存</el-button>
        <el-button @click="isShow = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'

export default {
  name: "Attr",
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      attrList: [], // 平台属性
      isShow: true, // 表格显示与隐藏
      attrInfo: {
        // 添加/修改属性
        attrName: "", // 属性名
        attrValueList: [], // 属性值
        categoryId: 0, // 三级分类id
        categoryLevel: 3, // 区分id等级
      },
    };
  },
  methods: {
    //自定义事件的回调
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
        //发请求获取平台的属性数据
        this.getAttrList();
      }
    },

    // 获取平台属性数据
    async getAttrList() {
      const { category1Id, category2Id, category3Id } = this;
      let result = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      );
      this.attrList = result.data;
    },

    // 添加属性值
    addAttrValue() {
      // 向属性值数组里面添加元素  响应式数组
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: '',
        flag: true, //用于切换查看模式和修改模式
      })
      // 自动聚焦
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length-1].focus()
      })
    },

    // 添加属性按钮
    addAttr() {
      this.isShow = false
      // 清除数据
      // 收集三级分类ID
      this.attrInfo = {
        attrName: "",
        attrValueList: [], 
        categoryId: this.category3Id,
        categoryLevel: 3,
      }
    },

    // 修改属性
    updateAttr(row) {
      this.isShow = false
      // 将选中的属性赋值给attrInfo,
      // 由于数据结构当中粗壮乃对象里面套数组，数组里面套对象，因此需要使用深拷贝解决
      // this.attrInfo = JSON.parse(JSON.stringify(row))
      this.attrInfo = cloneDeep(row)
      // 修改某一属性，将相应的属性值元素上加上flag这个标记
      this.attrInfo.attrValueList.forEach(item => {
        // 使用$set添加新属性，使其为响应式数据
        this.$set(item, 'flag', false)
      })
    },

    // 失去焦点事件，切换查看模式和修改模式
    toLook(row) {
      // 不能为空
      if (row.valueName.trim() == '') {
        this.$message.error('请输入正确的属性值')
        return;
      }
      // 新的不能与旧的重复
      let isRepat = this.attrInfo.attrValueList.some(item => {
        // row最新新增的属性值（数组的最后一项），需要在已有数组中去除
        if (row !== item) {
          return row.valueName == item.valueName
        }
      })
      if(isRepat) {
        this.$message.error('属性值重复')
        return
      }
      row.flag = false
    },

    // 点击span，变为编辑模式
    toEdit(row, index) {
      row.flag = true
      // 获取input节点，实现自动聚焦
      this.$nextTick(() => {
        this.$refs[index].focus()
      })
    },

    // 气泡确认框，删除属性
    deleteAttrValue(index) {
      this.attrInfo.attrValueList.splice(index,1)
    },

    // 保存按钮，添加属性或修改属性的保存
    async addOrUpdate() {
      // 整理参数：1.为空不能提交 2.flag不属于服务器字段
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(item => {
        if (item.valueName != '') {
          delete item.flag
          return true
        }
      })
      try {
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo)
        // 显示平台属性的信号量进行展示
        this.isShow = true
        this.$message.success('保存成功')
        this.getAttrList()
      } catch (error) {
      }
    }
  },
};
</script>

<style>
</style>