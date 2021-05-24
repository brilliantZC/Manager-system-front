<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('goodsorder_manage:goodsorder:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
        header-align="center" 
        align="center" 
        width="60" 
        type="index" 
        label="序号">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="主键"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="电话">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="地址">
      </el-table-column>
      <el-table-column
        prop="goods"
        header-align="center"
        align="center"
        label="选购商品">
      </el-table-column>
      <el-table-column
        prop="price"
        header-align="center"
        align="center"
        label="价格">
      </el-table-column>
      <el-table-column
        prop="timeStar"
        header-align="center"
        align="center"
        label="订单创建时间">
      </el-table-column>
      <el-table-column
        prop="timeStay"
        header-align="center"
        align="center"
        label="订单持续时间">
      </el-table-column>
      <el-table-column
        prop="zztdm"
        header-align="center"
        align="center"
        label="总状态代码"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="zztmc"
        header-align="center"
        align="center"
        label="订单状态">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="orderDetailHandle(scope.row.id)">订单详情</el-button>
          <el-button type="text" size="small" @click="jdsjconfirm(scope.row.id)" :disabled="qrjdconfirm(scope.row.zztdm)">接单配送</el-button>
          <el-button type="text" size="small" @click="ordersdconfirm(scope.row.id)" :disabled="ddsdconfirm(scope.row.zztdm)">订单送达</el-button>
          <el-button type="text" size="small" :disabled="ddwcconfirm(scope.row.zztdm)">订单完成</el-button>
          <!-- <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button> -->
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
    <!-- 确认接单并配送 -->
    <el-dialog
      title="确认接单并配送"
      :visible.sync="jdsjVisible"
      width="30%"
      center>
      <span>确认接受该订单？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="jdsjVisible = false">取 消</el-button>
        <el-button type="primary" @click="jdsjSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 骑手订单送达 -->
    <el-dialog
      title="确认接单已送达？"
      :visible.sync="ddsdVisible"
      width="30%"
      center>
      <span>确认该订单已送达？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="ddsdVisible = false">取 消</el-button>
        <el-button type="primary" @click="ddsdSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    <order-detail v-if="orderDetailVisible" ref="OrderDetail" @refreshDataList="getDataList"></order-detail>
  </div>
</template>

<script>
  import AddOrUpdate from './goodsorder-add-or-update'
  import OrderDetail from './orderDetail'
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
        orderDetailVisible: false,
        jdsjVisible: false,
        jdsjid: 0,
        ddsdVisible: false,
        ddsdid: 0
      }
    },
    components: {
      AddOrUpdate,
      OrderDetail
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/goodsorder_manage/goodsorder/list'),
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
      // 详情
      orderDetailHandle (id) {
        this.orderDetailVisible = true
        this.$nextTick(() => {
          this.$refs.OrderDetail.initdetail(id)
        })
      },
      // 确认接单状态控制
      qrjdconfirm (zztdm) {
        if (zztdm === 3) {
          return false
        } else {
          return true
        }
      },
      // 确认接单
      jdsjconfirm (id) {
        this.jdsjid = id
        this.jdsjVisible = true
      },
      jdsjSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/jdsjinfo/${this.jdsjid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '接单成功，请尽快前往商家取货',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.jdsjVisible = false
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
      // 订单送达状态控制
      ddsdconfirm (zztdm) {
        if (zztdm === 5) {
          return false
        } else {
          return true
        }
      },
      ordersdconfirm (id) {
        this.ddsdid = id
        this.ddsdVisible = true
      },
      ddsdSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/ddsdinfo/${this.ddsdid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '订单送达，等待用户确认！',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.ddsdVisible = false
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
      // 订单完成状态控制
      ddwcconfirm (zztdm) {
        if (zztdm === 7) {
          return false
        } else {
          return true
        }
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
            url: this.$http.adornUrl('/goodsorder_manage/goodsorder/delete'),
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
