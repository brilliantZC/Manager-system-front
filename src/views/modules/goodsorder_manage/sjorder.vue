<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('goodsorder_manage:excel')" type="primary" @click="downloadHandle()">导出</el-button>
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
          <el-button type="text" size="small" @click="confirmorder(scope.row.id)" :disabled="qrddconfirm(scope.row.zztdm)">确认接单</el-button>
          <el-button type="text" size="small" @click="confirmmade(scope.row.id)" :disabled="zzwcconfirm(scope.row.zztdm)">制作完成</el-button>
          <el-button type="text" size="small" @click="confirmchps(scope.row.id)" :disabled="chpsconfirm(scope.row.zztdm)">出货</el-button>
          <el-button type="text" size="small" :disabled="ddwcconfirm(scope.row.zztdm)">订单完成</el-button>
          <!--<el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button> -->
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
    <!-- 确认订单 -->
    <el-dialog
      title="确认订单"
      :visible.sync="QrOrderVisible"
      width="30%"
      center>
      <span>确认接受该订单？</span>
      <el-input v-model="intro" placeholder="请输入内容" type="textarea" :rows="5" style="width:100%;" maxlength="500"></el-input>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="QrOrderVisible = false">取 消</el-button>
        <el-button type="warning" @click="rejectOrder()">拒 绝</el-button>
        <el-button type="primary" @click="qrOrderSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 商品制作完成 -->
    <el-dialog
      title="商品制作完成"
      :visible.sync="zzwcGoodsVisible"
      width="30%"
      center>
      <span>商品制作完成，等待骑手接单！</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="zzwcGoodsVisible = false">取 消</el-button>
        <el-button type="primary" @click="zzwcGoodsSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 骑手取货并配送 -->
    <el-dialog
      title="骑手取货并配送"
      :visible.sync="chpsVisible"
      width="30%"
      center>
      <span>骑手取货，确认出货</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="chpsVisible = false">取 消</el-button>
        <el-button type="primary" @click="chpsSubmit()">确 定</el-button>
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
        intro: '',
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        orderDetailVisible: false,
        QrOrderVisible: false,
        qrjeid: 0,
        zzwcGoodsVisible: false,
        zzwcid: 0,
        chpsVisible: false,
        chpsid: 0
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
      // 确认订单   状态控制
      qrddconfirm (zztdm) {
        if (zztdm === 1) {
          return false
        } else {
          return true
        }
      },
      // 确认订单
      confirmorder (id) {
        this.qrjeid = id
        this.QrOrderVisible = true
      },
      // 拒绝订单
      rejectOrder () {
        this.$http({
          url: this.$http.adornUrl('/goodsorder_manage/goodsorder/rejectorderlist'),
          method: 'get',
          params: this.$http.adornParams({
            'intro': this.intro,
            'key': this.qrjeid
          })
        }).then(({data}) => {
          console.log(this.dataForm.intro)
          if (data && data.code === 0) {
            this.$message({
              message: '拒绝该订单',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.QrOrderVisible = false
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
      qrOrderSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/qrorderinfo/${this.qrjeid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '确认订单',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.QrOrderVisible = false
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
      // 制作完成 状态控制
      zzwcconfirm (zztdm) {
        if (zztdm === 2) {
          return false
        } else {
          return true
        }
      },
      // 制作完成，等待骑手接单
      confirmmade (id) {
        this.zzwcid = id
        this.zzwcGoodsVisible = true
      },
      zzwcGoodsSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/zzwcinfo/${this.zzwcid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '等待骑手接单配送',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.zzwcGoodsVisible = false
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
      // 出货成功，骑手配送 状态控制
      chpsconfirm (zztdm) {
        if (zztdm === 4) {
          return false
        } else {
          return true
        }
      },
      // 出货成功，骑手配送
      confirmchps (id) {
        this.chpsid = id
        this.chpsVisible = true
      },
      chpsSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/goodsorder_manage/goodsorder/chpsinfo/${this.chpsid}`),
          method: 'post',
          data: this.$http.adornData({
            'id': this.id
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '出货成功！ 骑手配送中...',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.chpsVisible = false
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
      // 详情
      orderDetailHandle (id) {
        this.orderDetailVisible = true
        this.$nextTick(() => {
          this.$refs.OrderDetail.initdetail(id)
        })
      },
      // 导出走后台方法
      downloadHandle () {
        this.$http({
          url: this.$http.adornUrl('/goodsorder_manage/excel/order'),
          method: 'post',
          responseType: 'arraybuffer',
          params: this.$http.adornParams({fileName: '历史成功订单信息表.xls'})
        })
        .then(({data}) => {
          if (data) {
            const blob = new Blob([data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' })
            let fileName = '历史成功订单信息表.xls'
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
