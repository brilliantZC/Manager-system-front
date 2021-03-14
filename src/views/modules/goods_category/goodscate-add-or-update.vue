<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="类别" prop="cateName">
      <el-input v-model="dataForm.cateName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="数量" prop="cateNum">
      <el-input v-model="dataForm.cateNum" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="cateIntro">
      <el-input v-model="dataForm.cateIntro" placeholder=""></el-input>
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
          cateName: '',
          cateNum: '',
          cateIntro: ''
        },
        dataRule: {
          cateName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          cateNum: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          cateIntro: [
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
              url: this.$http.adornUrl(`/goods_category/goodscate/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.cateName = data.goodsCate.cateName
                this.dataForm.cateNum = data.goodsCate.cateNum
                this.dataForm.cateIntro = data.goodsCate.cateIntro
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
              url: this.$http.adornUrl(`/goods_category/goodscate/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'cateName': this.dataForm.cateName,
                'cateNum': this.dataForm.cateNum,
                'cateIntro': this.dataForm.cateIntro
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
