<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('bill_manage:daybill:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="dayBillid"
        header-align="center"
        align="center"
        label="日期">
      </el-table-column>
      <el-table-column
        prop="dayWeek"
        header-align="center"
        align="center"
        label="星期">
      </el-table-column>
      <el-table-column
        prop="dayIncome"
        header-align="center"
        align="center"
        label="收入">
      </el-table-column>
      <el-table-column
        prop="dayOutcome"
        header-align="center"
        align="center"
        label="支出">
      </el-table-column>
      <el-table-column
        prop="dayPure"
        header-align="center"
        align="center"
        label="纯收入">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="showDayBill(scope.row.dayBillid)">详情</el-button>
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

    <!-- 点击详情后显示当日所有订单在el-dialog中 -->
    <el-dialog title="当日详情" :close-on-click-modal="false" :visible.sync="showvisible">
    <el-table :data="dataListShow" border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="billId"
        header-align="center"
        align="center"
        label="流水号">
      </el-table-column>
      <el-table-column
        prop="billName"
        header-align="center"
        align="center"
        label="内容">
      </el-table-column>
      <el-table-column
        prop="billAccount"
        header-align="center"
        align="center"
        label="收支">
      </el-table-column>
      <el-table-column
        prop="billInout"
        header-align="center"
        align="center"
        label="收支类型">
      </el-table-column>
    </el-table>
    <span slot="footer" class="dialog-footer">
      <el-button @click="showvisible = false">取消</el-button>
      <el-button type="primary" @click="showEnd()">确定</el-button>
    </span>
  </el-dialog>
  </div>
</template>

<script>
  import AddOrUpdate from './daybill-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataFormShow: {
          key: ''
        },
        dataListShow: [],
        dataList: [],
        showdataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        showvisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/bill_manage/daybill/list'),
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
      // 点击详情后显示当日所有订单
      showDayBill (id) {
        // 拿到该行的日期数据，然后再用它去找到所有匹配的数据放入showdataList中
        this.showvisible = true
        this.dataListLoading = true
        this.dataFormShow.key = id
        this.$http({
          url: this.$http.adornUrl('/bill_manage/bill/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataFormShow.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataListShow = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataListShow = []
            this.totalPage = 0
          }
          this.dataListLoading = false
          this.getDataList()
        })
        this.getDataList()
      },
      showEnd () {
        this.showvisible = false
        this.getDataList()
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
            url: this.$http.adornUrl('/bill_manage/daybill/delete'),
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