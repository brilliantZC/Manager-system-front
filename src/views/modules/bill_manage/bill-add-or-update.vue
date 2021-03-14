<template>
  <el-dialog
    :title="!dataForm.id ? '收支账单' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="流水号" prop="billId">
      <el-input v-model="dataForm.billId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="内容" prop="billName">
      <el-input v-model="dataForm.billName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="金额" prop="billAccount">
      <el-input v-model="dataForm.billAccount" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="类型" prop="billInout">
      <el-radio v-model="dataForm.billInout" label="1">收入</el-radio>
      <el-radio v-model="dataForm.billInout" label="2">支出</el-radio>
    </el-form-item>
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
          billId: '',
          billName: '',
          billAccount: '',
          billInout: '1'
        },
        dataRule: {
          billId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          billName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          billAccount: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          billInout: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/bill_manage/bill/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.billId = data.bill.billId
                this.dataForm.billName = data.bill.billName
                this.dataForm.billAccount = data.bill.billAccount
                this.dataForm.billInout = data.bill.billInout
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
              url: this.$http.adornUrl(`/bill_manage/bill/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'billId': this.dataForm.billId,
                'billName': this.dataForm.billName,
                'billAccount': this.dataForm.billAccount,
                'billInout': this.dataForm.billInout
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
