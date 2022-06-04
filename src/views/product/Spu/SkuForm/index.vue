<template>
  <div>
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="SPU名称">{{spu.spuName}}</el-form-item>
      <el-form-item label="SKU名称">
        <el-input v-model="skuInfo.skuName" placeholder="请输入sku名称"></el-input>
      </el-form-item>
      <el-form-item label="价格(元)">
        <el-input v-model="skuInfo.price" type="number" placeholder="请输入价格(元)"></el-input>
      </el-form-item>
      <el-form-item label="重量(千克)">
        <el-input v-model="skuInfo.weight" placeholder="请输入重量(千克)"></el-input>
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input
          v-model="skuInfo.skuDesc"
          type="textarea"
          rows="4"
          placeholder="请输入规格描述"
        ></el-input>
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form :inline="true" ref="form" :model="form" label-width="80px">
          <el-form-item :label="attr.attrName" v-for="attr in attrInfoList" :key="attr.id">
            <el-select v-model="attr.attrIdAndValueId" placeholder="请选择">
              <el-option :label="attrValue.valueName" :value="`${attr.id}:${attrValue.id}`" v-for="attrValue in attr.attrValueList" :key="attrValue.id"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form :inline="true" ref="form" :model="form" label-width="80px">
          <el-form-item :label="sale.saleAttrName" v-for="sale in spuSaleAttrList" :key="sale.id">
            <el-select v-model="sale.attrIdAndValueId" placeholder="请选择">
              <el-option :label="saleValue.saleAttrValueName" value="value" v-for="saleValue in sale.spuSaleAttrValueList" :key="saleValue.id"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table :data="spuImageList" style="width: 100%" border @selection-change="handleSelectionChange">
          <el-table-column prop="prop" type="selection" width="80">
          </el-table-column>
          <el-table-column prop="prop" label="图片" width="width">
            <template slot-scope="{row}">
              <img :src="row.imgUrl" alt="" style="width: 100px;height: 100px">
            </template>
          </el-table-column>
          <el-table-column prop="imgName" label="名称" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{row}">
              <el-button type="primary"  v-if="row.isDefault == 0" @click="changeDefault(row)">设为默认</el-button>
              <el-button v-else>默认</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "SkuForm",
  data() {
    return {
      spuImageList: [], // 图片数据
      spuSaleAttrList: [], // 销售属性
      attrInfoList: [], // 平台属性
      // 收集SKU数据
      skuInfo: {
        // 第一类，父组件给的数据
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        // 第二类，v-model收集的数据
        skuName: '', // sku名称
        price: '', // 价格
        weight: '', // 重量
        skuDesc: '', // 描述
        // 第三类，需要自己书写代码
        skuDefaultImg: '',  // 默认图片
        skuImageList: [  // 收集图片的字段
          // {
          //   id: 0,
          //   imgName: "string",
          //   imgUrl: "string",
          //   isDefault: "string",
          //   skuId: 0,
          //   spuImgId: 0,
          // },
        ], 
        skuAttrValueList: [  // 平台属性
          // {
          //   attrId: 0,
          //   valueId: 0,
          // },
        ],
        skuSaleAttrValueList: [  // 销售属性
          // {
          //   id: 0,
          //   saleAttrId: 0,
          //   saleAttrName: "string",
          //   saleAttrValueId: 0,
          //   saleAttrValueName: "string",
          //   skuId: 0,
          //   spuId: 0,
          // },
        ],       
      },
      spu: {},
      // 收集图片的数据字段:但是需要注意，收集的数据目前缺少isDefault字段，将来提交给服务器数据的时候，需要整理参数
      imageList:[]
    }
  },
  methods: {
    // 获取skuFrom数据
    async getData(category1Id, category2Id, spu) {
      // 收集父组件给予的数据
      this.skuInfo.category3Id = spu.category3Id
      this.skuInfo.spuId = spu.id
      this.skuInfo.tmId = spu.tmId
      this.spu = spu

      // 获取图片列表数据
      let imageRes = await this.$API.spu.reqSpuImageList(spu.id)
      let list = imageRes.data
      list.forEach(item => {
        item.isDefault = 0
      });
      this.spuImageList = list
      // 获取销售属性
      let saleRes = await this.$API.spu.reqSpuSaleAttrList(spu.id)
      this.spuSaleAttrList = saleRes.data
      // 获取平台属性
      let attrRes = await this.$API.spu.reqAttrInfoList(category1Id, category2Id, spu.category3Id)
      this.attrInfoList = attrRes.data
    },

    // table表格复选框按钮
    handleSelectionChange(params) {
      // 缺少isDefault字段
      this.imageList = params
    },

    // 排他操作
    changeDefault(row) {
      // 全部变为0
      this.spuImageList.forEach(item => {
        item.isDefault = 0
      })
      // 点击的变为1
      row.isDefault = 1
      // 收集默认图片的地址
      this.skuDefaultImg = row.imgUrl
    },

    // 取消按钮
    cancel() {
      this.$emit('changeScenes', 0)
      // 清除数据
      Object.assign(this._data, this.$options.data())
    },
    // 保存按钮
    save() {
      // 整理参数
      // 方式一：整理平台属性
      // const {attrInfoList, skuInfo} = this
      // let arr = []
      // attrInfoList.forEach(item => {
      //   if (item.attrIdAndValueId) {
      //     const [attrId, valueId] = item.attrIdAndValueId.split(':')
      //     let obj = {valueId, attrId}
      //     arr.push(obj)          
      //   }
      // })
      // skuInfo.skuAttrValueList = arr

      // 整理平台属性
      const {attrInfoList, skuInfo, spuSaleAttrList, imageList} = this
      skuInfo.skuAttrValueList = attrInfoList.reduce((prev, item) => {
        if(item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(':')
          prev.push({attrId, valueId})
        }
        return prev
      }, [])
      // 整理销售属性
      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((prev, item) => {
        if(item.attrIdAndValueId) {
          const [saleAttrId, saleAttrValueId] = item.attrIdAndValueId.split(':')
          prev.push({saleAttrId, saleAttrValueId})
        }
        return prev        
      }, [])
      // 整理图片数据
      skuInfo.skuImageList = imageList.map(item => {
        return {
          imgName: item.imgName,
          imgUrl: item.imgUrl,
          isDefault: item.isDefault,
          spuImgId: item.id
        }
      })
      // 发送请求
      this.$API.spu.reqAddSku(skuInfo)
      this.$message.success('添加SKU成功')
      this.$emit('changeScenes', 0)     
    }
  }
};
</script>

<style>
</style>