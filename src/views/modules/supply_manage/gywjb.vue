<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('supply_manage:gywjb:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('supply_manage:gywjb:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;border-bottom:1px solid rgba(30, 23, 32, 0.38);border-top:1px solid rgba(30, 23, 32, 0.38);border-left:1px solid rgba(30, 23, 32, 0.38);">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="主键"
        v-if="false">
      </el-table-column>
      <el-table-column header-align="center" align="center" width="70" type="index" label="序号"></el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="发布人">
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="手机号"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="地址"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="materialName"
        header-align="center"
        align="center"
        label="材料名">
      </el-table-column>
      <el-table-column
        prop="materialNum"
        header-align="center"
        align="center"
        label="材料数量">
      </el-table-column>
      <el-table-column
        prop="materialPrice"
        header-align="center"
        align="center"
        label="材料价格">
      </el-table-column>
      <el-table-column
        prop="intro"
        header-align="center"
        align="center"
        label="简介"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="ztdm"
        header-align="center"
        align="center"
        label="状态代码"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="ztmc"
        header-align="center"
        align="center"
        label="状态名称"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="ksri"
        header-align="center"
        align="center"
        label="开始日期">
      </el-table-column>
      <el-table-column
        prop="jsrq"
        header-align="center"
        align="center"
        label="结束日期">
      </el-table-column>
      <el-table-column
        prop="wjdz"
        header-align="center"
        align="center"
        label="文件地址"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="wjmc"
        header-align="center"
        align="center"
        label="文件名称"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="shfs"
        header-align="center"
        align="center"
        label="送货方式">
      </el-table-column>
      <el-table-column
        prop="wjlxmc"
        header-align="center"
        align="center"
        label="文件类型名称"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="wjlxdm"
        header-align="center"
        align="center"
        label="文件类型代码"
        v-if="false">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="Gysdetail(scope.row.id)">详情</el-button>
          <el-button type="text" size="small" @click="GhConfirm(scope.row.id)" :disabled="ghconfirm(scope.row.zztdm)">确认供货</el-button>
          <el-button type="text" size="small" @click="GhSuccess(scope.row.id,scope.row.uid)" :disabled="ghsuccess(scope.row.zztdm)">上传供货单据</el-button>
          <el-button type="text" size="small" :disabled="ghddsuccess(scope.row.zztdm)">完成交易</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <el-dialog
      title="提示"
      :visible.sync="GysghVisible"
      width="30%"
      center>
      <span>确认进行供货？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="GysghVisible = false">取 消</el-button>
        <el-button type="primary" @click="qrghsubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <gysdetail v-if="GysdetailVisible" ref="Gysdetail" @refreshDataList="getDataList"></gysdetail>
    <gysbillsc v-if="GysbillscVisible" ref="Gysbillsc" @refreshDataList="getDataList"></gysbillsc>
  </div>
</template>

<script>
  import Gysdetail from './gysdetail.vue'
  import AddOrUpdate from './gywjb-add-or-update'
  import Gysbillsc from './ghsbillsc.vue'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        GysdetailVisible: false,
        qrghid: 0,
        GysghVisible: false,
        GysbillscVisible: false
      }
    },
    components: {
      AddOrUpdate,
      Gysdetail,
      Gysbillsc
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/supply_manage/gywjb/gylist'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 供应商详情
      Gysdetail (id) {
        this.GysdetailVisible = true
        this.$nextTick(() => {
          this.$refs.Gysdetail.init(id)
        })
        this.getDataList()
      },
      // 确认供货 状态
      ghconfirm (zztdm) {
        if (zztdm === '3') {
          return false
        } else {
          return true
        }
      },
      ghddsuccess (zztdm) {
        if (zztdm === '6') {
          return false
        } else {
          return true
        }
      },
      // 供应商确认供货
      GhConfirm (id) {
        this.qrghid = id
        this.GysghVisible = true
      },
      qrghsubmit () {
        this.$http({
          url: this.$http.adornUrl(`/supply_manage/gywjb/qrinfo/${this.qrghid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '确认订单成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.GysghVisible = false
                this.$emit('refreshDataList')
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
        this.getDataList()
      },
      // 上传供货单据状态
      ghsuccess (zztdm) {
        if (zztdm === '4') {
          return false
        } else {
          return true
        }
      },
      // 上传供货单据
      GhSuccess (id, uid) {
        this.GysbillscVisible = true
        this.$nextTick(() => {
          this.$refs.Gysbillsc.init(id, uid)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/supply_manage/gywjb/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
<style lang="css" scoped>
/*加粗table线条 */
 .el-table >>> td{
     border-bottom:  1px solid rgba(30, 23, 32, 0.38);
     border-right:  1px solid rgba(30, 23, 32, 0.38);
  }
  .el-table >>> th{
    background-color: rgb(241, 243, 243);
    color: rgb(48, 47, 47);
    border-bottom:  1px solid rgba(30, 23, 32, 0.38);
    border-right:  1px solid rgba(30, 23, 32, 0.38);
  }
</style>