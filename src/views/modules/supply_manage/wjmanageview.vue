<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
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
          <el-button type="text" size="small" @click="Gysdetail(scope.row.id)">详情查看</el-button>
          <el-button type="text" size="small" @click="Gysbuy(scope.row.id)" :disabled="pdhandle(scope.row.zztdm)">选购</el-button>
          <el-button type="text" size="small" @click="Gyssuccess(scope.row.id)" :disabled="wcpdhandle(scope.row.zztdm)">确认供货完成</el-button>
          <el-button type="text" size="small" :disabled="wcddpdhandle(scope.row.zztdm)">订单完成</el-button>
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

    <!-- 弹窗, 详情查看 -->
     <gysdetail v-if="GysdetailVisible" ref="Gysdetail" @refreshDataList="getDataList"></gysdetail>
     <gysbuy v-if="GysbuyVisible" ref="Gysbuy" @refreshDataList="getDataList"></gysbuy>
     <gyssuccess v-if="GyssuccessVisible" ref="Gyssuccess" @refreshDataList="getDataList"></gyssuccess>
  </div>
</template>

<script>
  import Gysdetail from './gysdetail.vue'
  import Gysbuy from './gysbuy.vue'
  import Gyssuccess from './gyssuccess.vue'
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
        GysdetailVisible: false,
        GysbuyVisible: false,
        GyssuccessVisible: false
      }
    },
    components: {
      Gysdetail,
      Gyssuccess,
      Gysbuy
    },
    activated () {
      this.getDataList()
      this.pdhandle()
      this.wcpdhandle()
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
      // 是否可以采购
      pdhandle (zztdm) {
        if (zztdm === '1') {
          return false
        } else {
          return true
        }
      },
      wcpdhandle (zztdm) {
        if (zztdm === '5') {
          return false
        } else {
          return true
        }
      },
      wcddpdhandle (zztdm) {
        if (zztdm === '6') {
          return false
        } else {
          return true
        }
      },
      // 供应商详情
      Gysdetail (id) {
        this.GysdetailVisible = true
        this.$nextTick(() => {
          this.$refs.Gysdetail.init(id)
        })
      },
      // 管理员选购
      Gysbuy (id) {
        this.GysbuyVisible = true
        this.$nextTick(() => {
          this.$refs.Gysbuy.init(id)
        })
      },
      // 确认供货完成
      Gyssuccess (id) {
        this.GyssuccessVisible = true
        this.$nextTick(() => {
          this.$refs.Gyssuccess.init(id)
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