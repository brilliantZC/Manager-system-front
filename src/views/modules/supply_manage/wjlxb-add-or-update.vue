<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="进度名称" prop="jdmc">
      <el-input v-model="dataForm.jdmc" placeholder="进度名称"></el-input>
    </el-form-item>
    <el-form-item label="进度代码" prop="jddm">
      <el-input v-model="dataForm.jddm" placeholder="进度代码"></el-input>
    </el-form-item>
    <el-form-item label="类型名称" prop="wjlxmc">
      <el-input v-model="dataForm.wjlxmc" placeholder="类型名称"></el-input>
    </el-form-item>
    <el-form-item label="类型代码" prop="wjlxdm">
      <el-input v-model="dataForm.wjlxdm" placeholder="类型代码"></el-input>
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
          jdmc: '',
          jddm: '',
          wjlxmc: '',
          wjlxdm: ''
        },
        dataRule: {
          jdmc: [
            { required: true, message: '进度名称不能为空', trigger: 'blur' }
          ],
          jddm: [
            { required: true, message: '进度代码不能为空', trigger: 'blur' }
          ],
          wjlxmc: [
            { required: true, message: '类型名称不能为空', trigger: 'blur' }
          ],
          wjlxdm: [
            { required: true, message: '类型代码不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/supply_manage/wjlxb/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.jdmc = data.wjlxb.jdmc
                this.dataForm.jddm = data.wjlxb.jddm
                this.dataForm.wjlxmc = data.wjlxb.wjlxmc
                this.dataForm.wjlxdm = data.wjlxb.wjlxdm
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
              url: this.$http.adornUrl(`/supply_manage/wjlxb/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'jdmc': this.dataForm.jdmc,
                'jddm': this.dataForm.jddm,
                'wjlxmc': this.dataForm.wjlxmc,
                'wjlxdm': this.dataForm.wjlxdm
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
