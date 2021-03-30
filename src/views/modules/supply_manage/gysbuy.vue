<template>
  <el-dialog
  title="提示"
  :visible.sync="GysbuyVisible"
  width="30%"
  center>
  <span>确认选购该产品吗？</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="GysbuyVisible = false">取 消</el-button>
    <el-button type="primary" @click="submit()">确 定</el-button>
  </span>
</el-dialog>
</template>


<script>
  export default {
    data () {
      return {
        GysbuyVisible: false,
        id: 0
      }
    },
    methods: {
      init (id) {
        this.GysbuyVisible = true
        this.id = id
      },
      submit () {
        this.$http({
          url: this.$http.adornUrl(`/supply_manage/gywjb/xginfo/${this.id}`),
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
                this.GysbuyVisible = false
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