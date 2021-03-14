<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('bill_manage:monbill:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
      <!-- <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="主键">
      </el-table-column> -->
      <el-table-column
        prop="monYear"
        header-align="center"
        align="center"
        label="年份">
      </el-table-column>
      <el-table-column
        prop="monMon"
        header-align="center"
        align="center"
        label="月份">
      </el-table-column>
      <!-- <el-table-column
        prop="monBillid"
        header-align="center"
        align="center"
        label="日期">
      </el-table-column> -->
      <el-table-column
        prop="monIncome"
        header-align="center"
        align="center"
        label="月收入">
      </el-table-column>
      <el-table-column
        prop="monOutcome"
        header-align="center"
        align="center"
        label="月支出">
      </el-table-column>
      <el-table-column
        prop="monPure"
        header-align="center"
        align="center"
        label="月纯盈利">
      </el-table-column>
      <el-table-column
        prop="monIntro"
        header-align="center"
        align="center"
        label="备注">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="showMon(scope.row.monMon,scope.row.monYear)">详情</el-button>
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
    <el-dialog title="当月详情" :close-on-click-modal="false" :visible.sync="showmonvisible">
    <el-table
      :data="dataListMon"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
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
      </el-table>
      <span slot="footer" class="dialog-footer">
         <el-button @click="showmonvisible = false">取消</el-button>
         <el-button type="primary" @click="showMonEnd()">确定</el-button>
       </span>
  </el-dialog>
  </div>
</template>

<script>
  import AddOrUpdate from './monbill-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataFormMon: {
          key: '',
          key1: ''
        },
        dataList: [],
        dataListMon: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        showmonvisible: false
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
          url: this.$http.adornUrl('/bill_manage/monbill/list'),
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
            url: this.$http.adornUrl('/bill_manage/monbill/delete'),
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
      },
      showMon (mon, year) {
        this.showmonvisible = true
        this.dataListLoading = true
        this.dataFormMon.key = mon
        this.dataFormMon.key1 = year
        this.$http({
          url: this.$http.adornUrl('/bill_manage/daybill/listday'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataFormMon.key,
            'key1': this.dataFormMon.key1
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataListMon = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataListMon = []
            this.totalPage = 0
          }
          this.dataListLoading = false
          this.getDataList()
        })
      },
      showMonEnd () {
        this.showmonvisible = false
        this.getDataList()
      }
    }
  }
</script>
