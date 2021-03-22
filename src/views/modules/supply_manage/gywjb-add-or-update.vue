<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="发布人" prop="name">
      <el-input v-model="dataForm.name" placeholder="发布人"></el-input>
    </el-form-item>
    <el-form-item label="手机号" prop="phone">
      <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
    </el-form-item>
    <el-form-item label="地址" prop="address">
      <el-input v-model="dataForm.address" placeholder="地址"></el-input>
    </el-form-item>
    <el-form-item label="材料名" prop="materialName">
      <el-input v-model="dataForm.materialName" placeholder="材料名"></el-input>
    </el-form-item>
    <el-form-item label="材料数量" prop="materialNum">
      <el-input v-model="dataForm.materialNum" placeholder="材料数量"></el-input>
    </el-form-item>
    <el-form-item label="材料价格" prop="materialPrice">
      <el-input v-model="dataForm.materialPrice" placeholder="材料价格"></el-input>
    </el-form-item>
    <el-form-item label="简介" prop="intro">
      <el-input v-model="dataForm.intro" placeholder="简介"></el-input>
    </el-form-item>
    <el-form-item label="状态代码" prop="ztdm">
      <el-input v-model="dataForm.ztdm" placeholder="状态代码"></el-input>
    </el-form-item>
    <el-form-item label="状态名称" prop="ztmc">
      <el-input v-model="dataForm.ztmc" placeholder="状态名称"></el-input>
    </el-form-item>
    <el-form-item label="开始日期" prop="ksri">
      <el-input v-model="dataForm.ksri" placeholder="开始日期"></el-input>
    </el-form-item>
    <el-form-item label="结束日期" prop="jsrq">
      <el-input v-model="dataForm.jsrq" placeholder="结束日期"></el-input>
    </el-form-item>
    <el-form-item label="文件地址" prop="wjdz">
      <el-input v-model="dataForm.wjdz" placeholder="文件地址"></el-input>
    </el-form-item>
    <el-form-item label="文件名称" prop="wjmc">
      <el-input v-model="dataForm.wjmc" placeholder="文件名称"></el-input>
    </el-form-item>
    <el-form-item label="送货方式" prop="shfs">
      <el-input v-model="dataForm.shfs" placeholder="送货方式"></el-input>
    </el-form-item>
    <el-form-item label="文件类型名称" prop="wjlxmc">
      <el-input v-model="dataForm.wjlxmc" placeholder="文件类型名称"></el-input>
    </el-form-item>
    <el-form-item label="文件类型代码" prop="wjlxdm">
      <el-input v-model="dataForm.wjlxdm" placeholder="文件类型代码"></el-input>
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
          name: '',
          phone: '',
          address: '',
          materialName: '',
          materialNum: '',
          materialPrice: '',
          intro: '',
          ztdm: '',
          ztmc: '',
          ksri: '',
          jsrq: '',
          wjdz: '',
          wjmc: '',
          shfs: '',
          wjlxmc: '',
          wjlxdm: ''
        },
        dataRule: {
          name: [
            { required: true, message: '发布人不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '手机号不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '地址不能为空', trigger: 'blur' }
          ],
          materialName: [
            { required: true, message: '材料名不能为空', trigger: 'blur' }
          ],
          materialNum: [
            { required: true, message: '材料数量不能为空', trigger: 'blur' }
          ],
          materialPrice: [
            { required: true, message: '材料价格不能为空', trigger: 'blur' }
          ],
          intro: [
            { required: true, message: '简介不能为空', trigger: 'blur' }
          ],
          ztdm: [
            { required: true, message: '状态代码不能为空', trigger: 'blur' }
          ],
          ztmc: [
            { required: true, message: '状态名称不能为空', trigger: 'blur' }
          ],
          ksri: [
            { required: true, message: '开始日期不能为空', trigger: 'blur' }
          ],
          jsrq: [
            { required: true, message: '结束日期不能为空', trigger: 'blur' }
          ],
          wjdz: [
            { required: true, message: '文件地址不能为空', trigger: 'blur' }
          ],
          wjmc: [
            { required: true, message: '文件名称不能为空', trigger: 'blur' }
          ],
          shfs: [
            { required: true, message: '送货方式不能为空', trigger: 'blur' }
          ],
          wjlxmc: [
            { required: true, message: '文件类型名称不能为空', trigger: 'blur' }
          ],
          wjlxdm: [
            { required: true, message: '文件类型代码不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/supply_manage/gywjb/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.gywjb.name
                this.dataForm.phone = data.gywjb.phone
                this.dataForm.address = data.gywjb.address
                this.dataForm.materialName = data.gywjb.materialName
                this.dataForm.materialNum = data.gywjb.materialNum
                this.dataForm.materialPrice = data.gywjb.materialPrice
                this.dataForm.intro = data.gywjb.intro
                this.dataForm.ztdm = data.gywjb.ztdm
                this.dataForm.ztmc = data.gywjb.ztmc
                this.dataForm.ksri = data.gywjb.ksri
                this.dataForm.jsrq = data.gywjb.jsrq
                this.dataForm.wjdz = data.gywjb.wjdz
                this.dataForm.wjmc = data.gywjb.wjmc
                this.dataForm.shfs = data.gywjb.shfs
                this.dataForm.wjlxmc = data.gywjb.wjlxmc
                this.dataForm.wjlxdm = data.gywjb.wjlxdm
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
              url: this.$http.adornUrl(`/supply_manage/gywjb/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'materialName': this.dataForm.materialName,
                'materialNum': this.dataForm.materialNum,
                'materialPrice': this.dataForm.materialPrice,
                'intro': this.dataForm.intro,
                'ztdm': this.dataForm.ztdm,
                'ztmc': this.dataForm.ztmc,
                'ksri': this.dataForm.ksri,
                'jsrq': this.dataForm.jsrq,
                'wjdz': this.dataForm.wjdz,
                'wjmc': this.dataForm.wjmc,
                'shfs': this.dataForm.shfs,
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
