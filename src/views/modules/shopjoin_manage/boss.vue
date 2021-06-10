<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('shopjoin_manage:shopjoin:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
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
        label="负责人">
      </el-table-column>
      <el-table-column
        prop="phone"
        header-align="center"
        align="center"
        label="联系电话">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="预开店地址">
      </el-table-column>
      <el-table-column
        prop="money"
        header-align="center"
        align="center"
        label="资金成本">
      </el-table-column>
      <el-table-column
        prop="yyxk"
        header-align="center"
        align="center"
        label="营业许可"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="yyxkztdm"
        header-align="center"
        align="center"
        label="营业许可">
        <template slot-scope="scope">
            <span  v-if="scope.row.yyxkztdm==1" style="color:green">已上传</span>
            <span v-else style="color: red">未上传</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="wsxk"
        header-align="center"
        align="center"
        label="卫生许可"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="wsxkztdm"
        header-align="center"
        align="center"
        label="卫生许可">
        <template slot-scope="scope">
            <span  v-if="scope.row.wsxkztdm==1" style="color:green">已上传</span>
            <span v-else style="color: red">未上传</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="addressResult"
        header-align="center"
        align="center"
        label="实地考察结果">
        <template slot-scope="scope">
            <span  v-if="scope.row.addressResult=='通过'" style="color:green">考察通过</span>
            <span v-else-if="scope.row.addressResult==''" style="color: red">未考察</span>
            <span v-else style="color: red">{{scope.row.addressResult}},未通过！</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="grxd"
        header-align="center"
        align="center"
        label="个人信贷"
        v-if="false">
      </el-table-column>
      <el-table-column
        prop="grxdztdm"
        header-align="center"
        align="center"
        label="个人信贷">
        <template slot-scope="scope">
            <span  v-if="scope.row.grxdztdm==1" style="color:green">已上传</span>
            <span v-else style="color: red">未上传</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="tpResult"
        header-align="center"
        align="center"
        label="管理员投票结果">
        <template slot-scope="scope">
            <span  v-if="scope.row.tpResult=='通过'" style="color:green">通过</span>
            <span v-else style="color: red">未投票</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="finalResult"
        header-align="center"
        align="center"
        label="最终审核结果">
        <template slot-scope="scope">
            <span  v-if="scope.row.finalResult=='通过'" style="color:green">通过</span>
            <span v-else style="color: red">未最终审核</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="finalTime"
        header-align="center"
        align="center"
        label="审核时间"
        v-if="false">
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
        label="当前状态">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="JoinDetailHandle(scope.row.id)">详情</el-button>
          <el-button type="text" size="small" @click="shpassHandle(scope.row.id)" :disabled="shconfirm(scope.row.zztdm)">审核同意</el-button>
          <el-button type="text" size="small" @click="shnopassHandle(scope.row.id)" :disabled="shconfirm(scope.row.zztdm)">审核不同意</el-button>
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
    <el-dialog
      title="确定同意该加盟申请"
      :visible.sync="shpassVisible"
      width="30%"
      center>
      <span>确定同意？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="shpassVisible = false">取 消</el-button>
        <el-button type="primary" @click="shpassSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="确定不同意该加盟申请"
      :visible.sync="shnopassVisible"
      width="30%"
      center>
      <span>确定不同意？</span>
      <el-input v-model="finresult" placeholder="请输入驳回原因" type="textarea" :rows="5" style="width:100%;" maxlength="500"></el-input>
      <span slot="footer" class="dialog-footer">
        <el-button @click="shnopassVisible = false">取 消</el-button>
        <el-button type="primary" @click="shnopassSubmit()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 详情展示页 -->
    <JoinDetail v-if="detailVisible" ref="JoinDetail" @refreshDataList="getDataList"></JoinDetail>
  </div>
</template>

<script>
  import JoinDetail from './shopjoindetail'
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
        detailVisible: false,
        shpassVisible: false,
        shnopassVisible: false,
        shpassid: 0,
        shnopassid: 0,
        finresult: ''
      }
    },
    components: {
      JoinDetail
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/shopjoin_manage/shopjoin/list'),
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
      // 详情展示页
      JoinDetailHandle (id) {
        this.detailVisible = true
        this.$nextTick(() => {
          this.$refs.JoinDetail.init(id)
        })
      },
      shpassHandle (id) {
        this.shpassid = id
        this.shpassVisible = true
      },
      shnopassHandle (id) {
        this.shnopassid = id
        this.shnopassVisible = true
      },
      shpassSubmit () {
        this.$http({
          url: this.$http.adornUrl(`/shopjoin_manage/shopjoin/shpassinfo/${this.shpassid}`),
          method: 'post',
          data: this.$http.adornData()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '总部审核通过！',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.shpassVisible = false
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
      shnopassSubmit () {
        this.$http({
          url: this.$http.adornUrl('/shopjoin_manage/shopjoin/shnopasslist'),
          method: 'get',
          params: this.$http.adornParams({
            'result': this.finresult,
            'id': this.shnopassid
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '驳回成功！',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.shnopassVisible = false
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
      shconfirm (zztdm) {
        if (zztdm === 6) {
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
            url: this.$http.adornUrl('/shopjoin_manage/shopjoin/delete'),
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
