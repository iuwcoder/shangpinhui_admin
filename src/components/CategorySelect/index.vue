<template>
  <div>
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <el-form-item label="一级分类">
        <el-select v-model="formInline.category1Id" placeholder="请选择" @change="handle1" :disabled="!show">
          <el-option :label="c1.name" :value="c1.id" v-for="c1 in list1" :key="c1.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select v-model="formInline.category2Id" placeholder="请选择" @change="handle2" :disabled="!show">
          <el-option :label="c2.name" :value="c2.id" v-for="c2 in list2" :key="c2.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select v-model="formInline.category3Id" placeholder="请选择" @change="handle3" :disabled="!show">
          <el-option :label="c3.name" :value="c3.id" v-for="c3 in list3" :key="c3.id"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "CategorySelect",
  props: ['show'],
  data() {
    return {
      list1: [],
      list2: [],
      list3: [],
      formInline: {
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  methods: {
    // 获取一级分类
    async getCategory1List() {
      let result = await this.$API.attr.reqCategory1List();
      this.list1 = result.data

    },
    // 获取三级分类
    async getCategory3List() {
      await this.$API.attr.reqCategory3List()
    },

    // 一级分类变化后，获取二级分类列表
    async handle1() {
      this.list2 = []
      this.list3 = []
      this.formInline.category2Id = ''
      this.formInline.category3Id = ''
      const {category1Id} = this.formInline
      this.$emit('getCategoryId', {categoryId: category1Id, level: 1})
      let result = await this.$API.attr.reqCategory2List(category1Id)
      this.list2 = result.data
    },

    // 二级分类变化后，获取三级分类列表
    async handle2() {
      this.list3 = []
      this.formInline.category3Id = ''
      const {category2Id} = this.formInline
      this.$emit('getCategoryId', {categoryId: category2Id, level: 2})
      let result = await this.$API.attr.reqCategory3List(category2Id)
      this.list3 = result.data
    },

    handle3() {
      const {category3Id} = this.formInline
      this.$emit('getCategoryId', {categoryId: category3Id, level: 3})
    }
  },
  mounted() {
     this.getCategory1List();
  }
};
</script>

<style>
</style>