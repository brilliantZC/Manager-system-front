<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="文件类型" prop="wjlx">
      <el-input v-model="dataForm.wjlx" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="文件名称" prop="wjmc">
      <el-input v-model="dataForm.wjmc" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="文件类型代码" prop="wjlxdm">
      <el-input v-model="dataForm.wjlxdm" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="状态代码" prop="ztdm">
      <el-input v-model="dataForm.ztdm" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="状态名称" prop="ztmc">
      <el-input v-model="dataForm.ztmc" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="uid" prop="uid">
      <el-input v-model="dataForm.uid" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="文件地址" prop="wjdz">
      <el-input v-model="dataForm.wjdz" placeholder=""></el-input>
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
          wjlx: '',
          wjmc: '',
          wjlxdm: '',
          ztdm: '',
          ztmc: '',
          uid: '',
          wjdz: ''
        },
        dataRule: {
          wjlx: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          wjmc: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          wjlxdm: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          ztdm: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          ztmc: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          uid: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          wjdz: [
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
              url: this.$http.adornUrl(`/supply_manage/wjsave/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.wjlx = data.wjsave.wjlx
                this.dataForm.wjmc = data.wjsave.wjmc
                this.dataForm.wjlxdm = data.wjsave.wjlxdm
                this.dataForm.ztdm = data.wjsave.ztdm
                this.dataForm.ztmc = data.wjsave.ztmc
                this.dataForm.uid = data.wjsave.uid
                this.dataForm.wjdz = data.wjsave.wjdz
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
              url: this.$http.adornUrl(`/supply_manage/wjsave/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'wjlx': this.dataForm.wjlx,
                'wjmc': this.dataForm.wjmc,
                'wjlxdm': this.dataForm.wjlxdm,
                'ztdm': this.dataForm.ztdm,
                'ztmc': this.dataForm.ztmc,
                'uid': this.dataForm.uid,
                'wjdz': this.dataForm.wjdz
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
