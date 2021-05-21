<template>
  <el-dialog
    :title="订单详情"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-row>
      <el-col :span="12">
        <el-form-item label="姓名" prop="name">
          <el-input v-model="dataForm.name" placeholder="姓名" readonly></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="电话" prop="phone">
          <el-input v-model="dataForm.phone" placeholder="电话" readonly></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="地址" prop="address">
          <el-input v-model="dataForm.address" placeholder="地址" readonly></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="选购商品">
          <el-input v-model="dataForm.goods" placeholder="选购商品" readonly></el-input> 
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="价格" prop="price">
          <el-input v-model="dataForm.price" placeholder="价格" readonly></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="订单创建时间" prop="timeStar">
          <el-input v-model="dataForm.timeStar" placeholder="订单创建时间" readonly></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="24">
        <el-form-item label="订单持续时间" prop="timeStay">
          <el-input v-model="dataForm.timeStay" placeholder="订单持续时间" readonly></el-input>
        </el-form-item>
      </el-col>
    </el-row>
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
          timeStay: ''
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
        }
      }
    },
    methods: {
      initdetail (id) {
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
        this.visible = false
      }
    }
  }
</script>
