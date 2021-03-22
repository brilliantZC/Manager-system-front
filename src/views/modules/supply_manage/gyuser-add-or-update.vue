<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="gyname">
      <el-input v-model="dataForm.gyname" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gyphone">
      <el-input v-model="dataForm.gyphone" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gyaddress">
      <el-input v-model="dataForm.gyaddress" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gycount">
      <el-input v-model="dataForm.gycount" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="flag">
      <el-input v-model="dataForm.flag" placeholder=""></el-input>
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
          gyname: '',
          gyphone: '',
          gyaddress: '',
          gycount: '',
          flag: ''
        },
        dataRule: {
          gyname: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gyphone: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gyaddress: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gycount: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          flag: [
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
              url: this.$http.adornUrl(`/supply_manage/gyuser/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gyname = data.gyuser.gyname
                this.dataForm.gyphone = data.gyuser.gyphone
                this.dataForm.gyaddress = data.gyuser.gyaddress
                this.dataForm.gycount = data.gyuser.gycount
                this.dataForm.flag = data.gyuser.flag
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
              url: this.$http.adornUrl(`/supply_manage/gyuser/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gyname': this.dataForm.gyname,
                'gyphone': this.dataForm.gyphone,
                'gyaddress': this.dataForm.gyaddress,
                'gycount': this.dataForm.gycount,
                'flag': this.dataForm.flag
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
