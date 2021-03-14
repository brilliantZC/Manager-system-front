<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="日" prop="dayDay">
      <el-input v-model="dataForm.dayDay" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="月" prop="dayMon">
      <el-input v-model="dataForm.dayMon" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="年" prop="dayYear">
      <el-input v-model="dataForm.dayYear" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="星期" prop="dayWeek">
      <el-input v-model="dataForm.dayWeek" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="收入" prop="dayIncome">
      <el-input v-model="dataForm.dayIncome" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="支出" prop="dayOutcome">
      <el-input v-model="dataForm.dayOutcome" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="日期" prop="dayBillid">
      <el-input v-model="dataForm.dayBillid" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="纯收入" prop="dayPure">
      <el-input v-model="dataForm.dayPure" placeholder=""></el-input>
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
          dayDay: '',
          dayMon: '',
          dayYear: '',
          dayWeek: '',
          dayIncome: '',
          dayOutcome: '',
          dayBillid: '',
          dayPure: ''
        },
        dataRule: {
          dayDay: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayMon: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayYear: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayWeek: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayIncome: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayOutcome: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayBillid: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          dayPure: [
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
              url: this.$http.adornUrl(`/bill_manage/daybill/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.dayDay = data.dayBill.dayDay
                this.dataForm.dayMon = data.dayBill.dayMon
                this.dataForm.dayYear = data.dayBill.dayYear
                this.dataForm.dayWeek = data.dayBill.dayWeek
                this.dataForm.dayIncome = data.dayBill.dayIncome
                this.dataForm.dayOutcome = data.dayBill.dayOutcome
                this.dataForm.dayBillid = data.dayBill.dayBillid
                this.dataForm.dayPure = data.dayBill.dayPure
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
              url: this.$http.adornUrl(`/bill_manage/daybill/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'dayDay': this.dataForm.dayDay,
                'dayMon': this.dataForm.dayMon,
                'dayYear': this.dataForm.dayYear,
                'dayWeek': this.dataForm.dayWeek,
                'dayIncome': this.dataForm.dayIncome,
                'dayOutcome': this.dataForm.dayOutcome,
                'dayBillid': this.dataForm.dayBillid,
                'dayPure': this.dataForm.dayPure
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
