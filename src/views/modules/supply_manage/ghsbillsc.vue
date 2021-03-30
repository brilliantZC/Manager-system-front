<template>
  <el-dialog title="上传文件" 
  :close-on-click-modal="false" 
  :visible.sync="visible" 
  :append-to-body="true"
  :before-close="clearfile"
  width="40%">
    <el-form
      ref="dataForm"
      label-width="100px">
      <el-form-item label="文件上传类型:">
        <span>  .doc/.docx</span>
      </el-form-item>
      <el-form-item label="文件">
        <el-upload
          class="upload-demo"
          ref="upload"
          :data="uploadData"   
          :action="url"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :on-exceed="handleExceed"
          :file-list="fileList"
          :on-success="success"
          :before-upload="beforeAvatarUpload"
          :limit="1"
          :auto-upload="false">
          <el-button  size="small" type="primary" @change="preview($event)" >选择文件</el-button>
        </el-upload>
      </el-form-item>
    </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button  @click="clearfile()">取消</el-button>
        <el-button  type="primary" @click="submitUpload()">确定</el-button>
      </span>
    </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        id: 0,
        name: '',
        uid: 0,
        wjmc: '',
        wjdz: '',
        response: '',
        file: '',
        url: '',
        fileList: [],
        xmdata: [],
        dataList: [],
        uploadData: '',
        visible: false
      }
    },
    methods: {
      init (id, uid) {
        this.id = id
        this.uid = uid
        this.fileList = []
        this.visible = true
        this.url = this.$http.adornUrl(`/supply_manage/gywjb/ghfiles?token=${this.$cookie.get('token')}`)
        // 上传文件的时候向后台传参
        this.uploadData = {id: this.id, uid: this.uid}
      },
      // 手动上传文件列表
      submitUpload () {
        this.$nextTick(() => {
          this.$refs.upload.submit()
        })
      },
      // 清空文件列表
      clearfile () {
        this.fileList = []
        this.visible = false
      },
      // 上传成功时执行
      success (response, file, fileList) {
        this.wjmc = response.filename
        this.wjdz = response.Singlefile
        this.$http({
          // 上传成功后，向数据库存数据
          url: this.$http.adornUrl(`/supply_manage/gywjb/ghsave`),
          method: 'post',
          data: this.$http.adornData({
            wjmc: this.wjmc,
            wjdz: this.wjdz,
            wjlxmc: this.wjlxmc,
            name: this.name,
            wjlxdm: this.wjlxdm,
            ztdm: 1,
            uid: this.uid
          })
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$emit('tellFatherName')
            // 上传成功后，子组件调用父组件的函数，实现实时刷新数据
            this.$message({
              message: '操作成功',
              type: 'success',
              // 显示类型
              duration: 1500,
              // 显示的持续时间
              onClose: () => {
                // 关闭弹窗
                this.visible = false
                this.fileList = []
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      },
      // 文件列表移除文件时的函数
      handleRemove (file, fileList) {
        console.log(file, fileList)
      },
      // 点击文件列表中已上传的文件时的函数
      handlePreview (file) {
        console.log(file)
      },
      // 文件超出个数限制时的函数
      handleExceed (files, fileList) {
        this.$message.warning(`当前限制只能上传1个文件，列表已有 ${files.length} 个文件`)
      },
      beforeAvatarUpload (file) {
        var testmsg = file.name.substring(file.name.lastIndexOf('.') + 1)
        const extension = testmsg === 'doc' || testmsg === 'docx'
        if (!extension) {
          this.$message({
            message: '上传文件只能是 doc、docx!',
            type: 'warning'
          })
        }
        return extension
      }
    }
  }
</script>

      