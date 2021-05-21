<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="电话" prop="phone">
      <el-input v-model="dataForm.phone" placeholder="电话"></el-input>
    </el-form-item>
    <el-form-item label="地址" prop="address">
      <el-input v-model="dataForm.address" placeholder="地址"></el-input>
    </el-form-item>
    <el-form-item label="选购商品">
      <el-cascader
        v-model="value"
        :options="options"
        @change="handleChange">
     </el-cascader>
      <!-- <el-input v-model="dataForm.goods" placeholder="选购商品"></el-input> -->
    </el-form-item>
   <!-- <el-form-item label="价格" prop="price">
      <el-input v-model="dataForm.price" placeholder="价格"></el-input>
    </el-form-item>
    <el-form-item label="订单创建时间" prop="timeStar">
      <el-input v-model="dataForm.timeStar" placeholder="订单创建时间"></el-input>
    </el-form-item>
    <el-form-item label="订单持续时间" prop="timeStay">
      <el-input v-model="dataForm.timeStay" placeholder="订单持续时间"></el-input>
    </el-form-item>
    <el-form-item label="总状态代码" prop="zztdm">
      <el-input v-model="dataForm.zztdm" placeholder="总状态代码"></el-input>
    </el-form-item>
    <el-form-item label="总状态名称" prop="zztmc">
      <el-input v-model="dataForm.zztmc" placeholder="总状态名称"></el-input>
    </el-form-item>-->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          name: '',
          phone: '',
          address: '',
          goods: '',
          price: '',
          timeStar: '',
          timeStay: '',
          zztdm: '',
          zztmc: ''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '电话不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '地址不能为空', trigger: 'blur' }
          ],
          goods: [
            { required: true, message: '选购商品不能为空', trigger: 'blur' }
          ]
        },
        value: [],
        options: [{
          value: '营养肉粥',
          label: '营养肉粥',
          children: [{
            value: '瘦肉青菜粥',
            label: '瘦肉青菜粥'
          }, {
            value: '番茄瘦肉粥',
            label: '番茄瘦肉粥'
          }, {
            value: '鸡肉玉米粥',
            label: '鸡肉玉米粥'
          }, {
            value: '山药瘦肉粥',
            label: '山药瘦肉粥'
          }, {
            value: '番茄牛肉粥',
            label: '番茄牛肉粥'
          }, {
            value: '皮蛋牛肉粥',
            label: '皮蛋牛肉粥'
          }]
        }, {
          value: '全素粥类',
          label: '全素粥类',
          children: [{
            value: '玉米青菜粥',
            label: '玉米青菜粥'
          }, {
            value: '清火白粥',
            label: '清火白粥'
          }, {
            value: '玉米南瓜粥',
            label: '玉米南瓜粥'
          }, {
            value: '山药青菜粥',
            label: '山药青菜粥'
          }]
        }, {
          value: '海鲜粥类',
          label: '海鲜粥类',
          children: [{
            value: '干贝瘦肉粥',
            label: '干贝瘦肉粥'
          }, {
            value: '干贝鱼片粥',
            label: '干贝鱼片粥'
          }, {
            value: '干贝虾仁粥',
            label: '干贝虾仁粥'
          }, {
            value: '虾仁玉米粥',
            label: '虾仁玉米粥'
          }, {
            value: '生滚鱼片粥',
            label: '生滚鱼片粥'
          }]
        }, {
          value: '曼玲蒸品',
          label: '曼玲蒸品',
          children: [{
            value: '葱香大肉包（2个）',
            label: '葱香大肉包（2个）'
          }, {
            value: '灌汤小笼包',
            label: '灌汤小笼包'
          }, {
            value: '江南桂花糕',
            label: '江南桂花糕'
          }, {
            value: '古法奶香馒头',
            label: '古法奶香馒头'
          }]
        }, {
          value: '曼玲甜粥',
          label: '曼玲甜粥',
          children: [{
            value: '玉米南瓜粥',
            label: '玉米南瓜粥'
          }, {
            value: '燕麦牛奶粥',
            label: '燕麦牛奶粥'
          }, {
            value: '黑米粥',
            label: '黑米粥'
          }, {
            value: '红枣枸杞粥',
            label: '红枣枸杞粥'
          }]
        }, {
          value: '曼玲点心',
          label: '曼玲点心',
          children: [{
            value: '火山石烤肠',
            label: '火山石烤肠'
          }, {
            value: '香芋南瓜园（6个）',
            label: '香芋南瓜园（6个）'
          }, {
            value: '春卷（7个）',
            label: '春卷（7个）'
          }, {
            value: '牛肉馅饼（圆葱）',
            label: '牛肉馅饼（圆葱）'
          }]
        }, {
          value: '精品小菜',
          label: '精品小菜',
          children: [{
            value: '咸鸭蛋',
            label: '咸鸭蛋'
          }, {
            value: '榨菜丝',
            label: '榨菜丝'
          }]
        }, {
          value: '曼玲饮品',
          label: '曼玲饮品',
          children: [{
            value: '可口可乐350ml',
            label: '可口可乐350ml'
          }, {
            value: '营养快线',
            label: '营养快线'
          }]
        }
        ]
      }
    },
    methods: {
      handleChange (value) {
        console.log(value)
      },
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.goodsorder.name
                this.dataForm.phone = data.goodsorder.phone
                this.dataForm.address = data.goodsorder.address
                this.dataForm.goods = data.goodsorder.goods
                this.dataForm.price = data.goodsorder.price
                this.dataForm.timeStar = data.goodsorder.timeStar
                this.dataForm.timeStay = data.goodsorder.timeStay
                this.dataForm.zztdm = data.goodsorder.zztdm
                this.dataForm.zztmc = data.goodsorder.zztmc
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/${!this.dataForm.id ? 'uordersave' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'goods': this.value[1],
                'price': this.dataForm.price,
                'timeStar': this.dataForm.timeStar,
                'timeStay': this.dataForm.timeStay,
                'zztdm': this.dataForm.zztdm,
                'zztmc': this.dataForm.zztmc
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
