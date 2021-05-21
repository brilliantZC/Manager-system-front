<template>
  <div class="mod-config">
    <el-tabs v-model="activeName" @tab-click="handleClick" type="card"> 
      <el-tab-pane label="日收支表" name="first" lazy>
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
            <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button v-if="isAuth('bill_manage:excel')" type="primary" @click="downloadHandle()">导出</el-button>
            <el-button v-if="isAuth('bill_manage:daybill:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
          </el-form-item>
        </el-form>
        <el-table
          :data="dataList"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;border-bottom: 1px solid rgba(30, 23, 32, 0.38);border-top: 1px solid rgba(30, 23, 32, 0.38);border-left: 1px solid rgba(30, 23, 32, 0.38);">
          <el-table-column
            type="selection"
            header-align="center"
            align="center"
            width="50">
          </el-table-column>
          <el-table-column 
            header-align="center" 
            align="center" 
            width="60" 
            type="index" 
            label="序号">
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
      </el-tab-pane>
    <el-tab-pane label="日收支图" name="second" lazy>
      <el-card>
        <el-form :inline="true" >
          <el-form-item label="日期" prop="value">
            <el-date-picker v-model="value" align="right" type="date" placeholder="选择日期" value-format="yyyyMMdd" format="yyyyMMdd" clearable>
          </el-date-picker>
          </el-form-item>
        <el-form-item>
          <el-button @click="initChartPie()">查询</el-button>
        </el-form-item>
        </el-form>
          <div id="J_chartPieBox" style="width:1000px; height:500px" class="chart-box"></div>
        </el-card>
        <p></p>
        <el-card>
           <div id="J_chartBarBox" style="width:1000px; height:500px" class="chart-box"></div>
        </el-card>
    </el-tab-pane>
    </el-tabs>
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
  import echarts from 'echarts'
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
        showvisible: false,
        activeName: 'first',
        value: '',
        chartPie: null,
        charbar: null,
        mname: [],
        mnum: [],
        zname: [],
        znum: []
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      if (this.chartBar) {
        this.chartBar.resize()
      }
      if (this.chartPie) {
        this.chartPie.resize()
      }
      this.getDataList()
      this.initChartBar()
      this.initChartPie()
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
      handleClick (tab, event) {
        switch (tab.name) {
          case 'first': this.getDataList(); break
          case 'second': this.initChartBar(); break
        }
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
     // 导出走后台方法
      downloadHandle () {
        this.$http({
          url: this.$http.adornUrl('/bill_manage/excel/daybill'),
          method: 'post',
          responseType: 'arraybuffer',
          params: this.$http.adornParams({fileName: '日收支信息表.xls'})
        })
        .then(({data}) => {
          if (data) {
            const blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
            let fileName = '日收支信息表.xls'
            if ('download' in document.createElement('a')) {
              let objectUrl = URL.createObjectURL(blob)
              let downEle = document.createElement('a')
              downEle.href = objectUrl
              downEle.setAttribute('download', fileName)
              document.body.appendChild(downEle)
              downEle.click()
            } else {
              navigator.msSaveBlob(blob, fileName)
            }
            this.getDataList()
          }
        })
      },
      initChartPie () {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl('/bill_manage/daybill/countinout'),
            method: 'get',
            params: this.$http.adornParams({billdate: this.value})
          }).then(({ data }) => {
            var option = {
              title: {
                text: data.billdate + '收支图',
                // subtext: '纯属虚构',
                x: 'center'
              },
              tooltip: {
                trigger: 'item',
                formatter: '{a} <br/>{b} : {c} ({d}%)'
              },
              legend: {
                orient: 'vertical',
                left: 'left',
                data: data.mname
              },
              series: [
                {
                  name: '金额',
                  type: 'pie',
                  radius: '55%',
                  center: ['50%', '60%'],
                  data: data.mnum,
                  itemStyle: {
                    emphasis: {
                      shadowBlur: 10,
                      shadowOffsetX: 0,
                      shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                  }
                }
              ]
            }
            this.chartBar = echarts.init(document.getElementById('J_chartPieBox'))
            this.chartBar.setOption(option)
            window.addEventListener('resize', () => {
              this.chartBar.resize()
            })
          })
        })
      },
       // 柱状图
      initChartBar () {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl('/bill_manage/daybill/daybilllist'),
            method: 'get'
          }).then(({ data }) => {
            var option = {
              title: {
                text: '日盈利柱状图',
                left: 'center'
              },
              tooltip: {
                trigger: 'axis',
                axisPointer: {
                  type: 'shadow'
                }
              },
              grid: {
                bottom: 90
              },
              dataZoom: [{
                type: 'inside'
              }, {
                type: 'slider'
              }],
              xAxis: {
                data: data.zname,
                silent: false,
                splitLine: {
                  show: false
                },
                splitArea: {
                  show: false
                },
                axisLabel: {
                  interval: 0,
                  rotate: 20, // 20度角倾斜显示(***这里是关键)
                  textStyle: {
                    color: '#000000'
                  }
                }
              },
              yAxis: {
                splitArea: {
                  show: false
                }
              },
              series: [{
                type: 'bar',
                data: data.znum,
                // Set `large` for large data amount
                large: true,
                // 柱图宽度
                barWidth: 40
              }]
            }
            this.chartBar = echarts.init(document.getElementById('J_chartBarBox'))
            this.chartBar.setOption(option)
            window.addEventListener('resize', () => {
              this.chartBar.resize()
            })
          })
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