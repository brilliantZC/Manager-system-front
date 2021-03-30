<template>
  <el-dialog
  title="提示"
  :visible.sync="GyssuccessVisible"
  width="30%"
  center>
  <span>确认已收到货物，并支付？</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="GyssuccessVisible = false">取 消</el-button>
    <el-button type="primary" @click="submit()">确 定</el-button>
  </span>
</el-dialog>
</template>


<script>
  export default {
    data () {
      return {
        GyssuccessVisible: false,
        id: 0
      }
    },
    methods: {
      init (id) {
        this.GyssuccessVisible = true
        this.id = id
      },
      submit () {
        this.$http({
          url: this.$http.adornUrl(`/supply_manage/gywjb/cginfo/${this.id}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '选购成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.GyssuccessVisible = false
                this.$emit('refreshDataList')
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }
    }
  }
</script>