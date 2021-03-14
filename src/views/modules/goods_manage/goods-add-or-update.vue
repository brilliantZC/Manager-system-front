<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="商品名" prop="gname">
      <el-input v-model="dataForm.gname" placeholder="商品名"></el-input>
    </el-form-item>
    <el-form-item label="商品价格" prop="gprice">
      <el-input v-model="dataForm.gprice" placeholder="商品价格"></el-input>
    </el-form-item>
    <el-form-item label="商品数量" prop="gcount">
      <el-input v-model="dataForm.gcount" placeholder="商品数量"></el-input>
    </el-form-item>
    <el-form-item label="商品图片" prop="gimage">
      <el-input v-model="dataForm.gimage" placeholder="商品图片"></el-input>
    </el-form-item>
    <el-form-item label="类别" prop="gintro">
       <el-select v-model="dataForm.gintro" clearable placeholder="请选择">
         <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
       </el-select>
      <!-- <el-input v-model="dataForm.gintro" placeholder="商品描述"></el-input> -->
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
          gname: '',
          gprice: '',
          gcount: '',
          gimage: '',
          gintro: ''
        },
        dataRule: {
          gname: [
            { required: true, message: '商品名不能为空', trigger: 'blur' }
          ],
          gprice: [
            { required: true, message: '商品价格不能为空', trigger: 'blur' }
          ],
          gcount: [
            { required: true, message: '商品数量不能为空', trigger: 'blur' }
          ],
          gimage: [
            { required: true, message: '商品图片不能为空', trigger: 'blur' }
          ],
          gintro: [
            { required: true, message: '商品描述不能为空', trigger: 'blur' }
          ]
        },

        options: [{
          value: '营养肉粥',
          label: '营养肉粥'
        }, {
          value: '海鲜粥类',
          label: '海鲜粥类'
        }, {
          value: '全素粥类',
          label: '全素粥类'
        }, {
          value: '曼玲甜粥',
          label: '曼玲甜粥'
        }, {
          value: '曼玲蒸品',
          label: '曼玲蒸品'
        }, {
          value: '曼玲点心',
          label: '曼玲点心'
        }, {
          value: '曼玲饮品',
          label: '曼玲饮品'
        }, {
          value: '精品小菜',
          label: '精品小菜'
        }],
        value: ''
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
              url: this.$http.adornUrl(`/goods_manage/goods/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gname = data.goods.gname
                this.dataForm.gprice = data.goods.gprice
                this.dataForm.gcount = data.goods.gcount
                this.dataForm.gimage = data.goods.gimage
                this.dataForm.gintro = data.goods.gintro
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
              url: this.$http.adornUrl(`/goods_manage/goods/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gname': this.dataForm.gname,
                'gprice': this.dataForm.gprice,
                'gcount': this.dataForm.gcount,
                'gimage': this.dataForm.gimage,
                'gintro': this.dataForm.gintro
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
