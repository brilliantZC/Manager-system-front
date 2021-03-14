<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="年份" prop="monYear">
      <el-input v-model="dataForm.monYear" placeholder="" :disabled="true"></el-input>
    </el-form-item>
    <el-form-item label="月份" prop="monMon">
      <el-input v-model="dataForm.monMon" placeholder="" :disabled="true"></el-input>
    </el-form-item>
    <el-form-item label="月收入" prop="monIncome">
      <el-input v-model="dataForm.monIncome" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="月支出" prop="monOutcome">
      <el-input v-model="dataForm.monOutcome" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="月纯盈利" prop="monPure">
      <el-input v-model="dataForm.monPure" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="monIntro">
      <el-input v-model="dataForm.monIntro" placeholder=""></el-input>
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
          monMon: '',
          monYear: '',
          monBillid: '',
          monIncome: '',
          monOutcome: '',
          monPure: '',
          monIntro: ''
        },
        dataRule: {
          monMon: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          monYear: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          monBillid: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          monIncome: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          monOutcome: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          monPure: [
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
              url: this.$http.adornUrl(`/bill_manage/monbill/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.monMon = data.monBill.monMon
                this.dataForm.monYear = data.monBill.monYear
                this.dataForm.monBillid = data.monBill.monBillid
                this.dataForm.monIncome = data.monBill.monIncome
                this.dataForm.monOutcome = data.monBill.monOutcome
                this.dataForm.monPure = data.monBill.monPure
                this.dataForm.monIntro = data.monBill.monIntro
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
              url: this.$http.adornUrl(`/bill_manage/monbill/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'monMon': this.dataForm.monMon,
                'monYear': this.dataForm.monYear,
                'monBillid': this.dataForm.monBillid,
                'monIncome': this.dataForm.monIncome,
                'monOutcome': this.dataForm.monOutcome,
                'monPure': this.dataForm.monPure,
                'monIntro': this.dataForm.monIntro
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
