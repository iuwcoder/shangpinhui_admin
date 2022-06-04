<template>
  <div>
    <el-form ref="form" :model="spu" label-width="80px">
      <el-form-item label="SPU名称">
        <el-input v-model="spu.spuName" placeholder="SPU名称"></el-input>
      </el-form-item>
      <el-form-item label="品牌">
        <el-select v-model="spu.tmId" placeholder="请选择品牌">
          <el-option
            :label="tm.tmName"
            :value="tm.id"
            v-for="tm in tradeMarkList"
            :key="tm.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          v-model="spu.description"
          type="textarea"
          rows="4"
          placeholder="请输入描述"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片">
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handlerSuccess"
          :file-list="spuImageList"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select v-model="attrIdAndName" :placeholder="`还有${unSelectSale.length}未选择`">
          <el-option :label="unSelect.name" :value="`${unSelect.id}:${unSelect.name}`" v-for="unSelect in unSelectSale" :key="unSelect.id"></el-option>
        </el-select>
        <el-button type="primary" icon="el-icon-plus" style="margin-left: 10px" :disabled="!attrIdAndName" @click="addSaleAttr">添加销售属性</el-button>
        <el-table :data="spu.spuSaleAttrList" style="width: 100%; margin-top: 10px" border>
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="saleAttrName" label="属性名" width="200">
          </el-table-column>
          <el-table-column prop="prop" label="属性值名称列表" width="width">
            <template slot-scope="{ row }">
              <el-tag :key="tag" v-for="(tag, index) in row.spuSaleAttrValueList" closable :disable-transitions="false" @close="row.spuSaleAttrValueList.splice(index,1)">
                {{ tag.saleAttrValueName }}</el-tag>
              <el-input
                class="input-new-tag"
                v-if="row.inputVisible"
                v-model="row.inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm(row)"
                @blur="handleInputConfirm(row)"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="showInput(row)"
                >添加</el-button
              >
            </template>
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="150">
            <template slot-scope="{$index}">
              <el-button type="danger" icon="el-icon-delete" @click="spu.spuSaleAttrList.splice($index,1)"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateSpu">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SpuForm",
  data() {
    return {
      dialogImageUrl: "",
      dialogVisible: false,
      //spu属性初始化的时候是一个空的对象,在修改SPU的时候，会想服务器发请求，返回SPU信息（对象），在修改的时候可以利用服务器返回的这个对象收集最新的数据提交给服务器
      //添加SPU，如果是添加SPU的时候并没有向服务器发请求，数据收集到哪里呀[SPU]，收集数据的时候有哪些字段呀，看文档
      spu: {
        category3Id: 0, //三级分类的id
        description: "", //描述
        spuName: "", //spu名称
        tmId: "", //品牌的id
        spuImageList: [
          //收集SPU图片的信息
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   spuId: 0,
          // },
        ],
        spuSaleAttrList: [
          //平台属性与属性值收集
          // {
          //   baseSaleAttrId: 0,
          //   id: 0,
          //   saleAttrName: "string",
          //   spuId: 0,
          //   spuSaleAttrValueList: [
          //     {
          //       baseSaleAttrId: 0,
          //       id: 0,
          //       isChecked: "string",
          //       saleAttrName: "string",
          //       saleAttrValueName: "string",
          //       spuId: 0,
          //     },
          //   ],
          // },
        ],
      },
      tradeMarkList: [], // 品牌信息
      spuImageList: [], // spu图片
      saleAttrList: [], // 平台全部的销售属性
      attrIdAndName: '', // 收集未选择的销售属性的id
      inputVisible: false,
      inputValue: "",
    };
  },
  computed: {
    //计算未选择的销售属性
    unSelectSale() {
      // 整个平台的销售属性共三个 ---saleAttrList
      // 当前SPU拥有属于自己的销售属性spu.spuSaleAttrList
      let result = this.saleAttrList.filter(item => {
        return this.spu.spuSaleAttrList.every(item1 => {
          return item.name != item1.saleAttrName
        })
      })
      return result
    }
  },
  methods: {
    // 删除照片
    handleRemove(file, fileList) {
      // 收集照片墙图片数据
      this.spuImageList = fileList;
    },
    // 预览照片
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 图片上传成功
    handlerSuccess(response, file, fileList) {
      this.spuImageList = fileList;
    },

    // 初始化数据
    async initSpuData(spu) {
      // 获取spu信息的数据
      let spuResult = await this.$API.spu.reqSpu(spu.id);
      this.spu = spuResult.data;
      // 获取品牌的信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      this.tradeMarkList = tradeMarkResult.data;
      // 获取spu图片的数据
      let spuImageResult = await this.$API.spu.reqSpuImageList(spu.id);
      let listArr = spuImageResult.data;
      // 由于照片墙显示图片的数据有name和url字段，需要修改
      listArr.forEach((item) => {
        item.name = item.imgName;
        item.url = item.imgUrl;
      });
      this.spuImageList = listArr;
      // 获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      this.saleAttrList = saleResult.data;
    },

    // 添加新的销售属性
    addSaleAttr() {
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndName.split(':')
      // 向SPU对象数组里面添加新的销售属性
      let newSaleAttr = {baseSaleAttrId, saleAttrName,spuSaleAttrValueList:[]}
      this.spu.spuSaleAttrList.push(newSaleAttr)
      this.attrIdAndName = ''
    },
    // 销售属性内 添加按钮的回调
    showInput(row) {
      // 挂载在销售属性身上恶的响应式数据inputVisible,控制button与input切换
      this.$set(row, 'inputVisible', true)
      // inputValue收集新增的销售属性值
      this.$set(row, 'inputValue', '')
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    // 失去焦点事件
    handleInputConfirm(row) {
      // 解构出销售属性里面的数据
      const {baseSaleAttrId, inputValue} = row
      if (inputValue.trim() == '') {
        this.$message.error('属性值不能为空')
        return
      }
      // 属性值不能重复
      let result = row.spuSaleAttrValueList.every(item => item.saleAttrValueName != inputValue)
      // true不重复，false重复
      if(!result) {
        this.$message.error('属性值不能重复')
        return
      }
      // 新增的销售属性值
      let newSaleAttrValue = {baseSaleAttrId,saleAttrValueName:inputValue}
      row.spuSaleAttrValueList.push(newSaleAttrValue)
      row.inputVisible = false;
    },

    // 保存按钮  添加/修改SPU
    async addOrUpdateSpu() {
      // 处理照片墙数据，需要携带imgName imgUrl字段
      this.spu.spuImageList = this.spuImageList.map(item => {
        return {
          imgName: item.name,
          imgUrl: (item.response&&item.response.data) || item.url
        }
      })
      await this.$API.spu.reqAddOrUpdateSpu(this.spu)
      this.$message.success('保存成功')
      // 切换场景，清除数据
      this.cancel()
    },

    // 取消按钮
    cancel() {
      this.$emit('changeScene', 0)
      // 清理数据
      // Object.assign合并对象，this._data组件实例 可以操作data里面的响应式数据
      // this.$options 可以获取配置对象，配置对象的data函数执行，返回的响应式数据为空
      Object.assign(this._data, this.$options.data())
    },

    // 点击添加SPU按钮的时候，发送请求
    async addSpuData(category3Id) {
      // 收集三级分类ID
      this.spu.category3Id = category3Id
      // 获取品牌的信息
      let tradeMarkResult = await this.$API.spu.reqTradeMarkList();
      this.tradeMarkList = tradeMarkResult.data;
      // 获取平台全部的销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      this.saleAttrList = saleResult.data;
    }
  },
};
</script>

<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>