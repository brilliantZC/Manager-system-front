<template>
  <div>
    <el-dialog
      title='申请详情'
      :close-on-click-modal="false"
      :visible.sync="visible">
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-collapse v-model="activeNames">
          <el-collapse-item title="基本信息" name="1">
            <el-row>
              <el-col :span="12">
                <el-form-item label="负责人" prop="name">
                  <el-input v-model="dataForm.name" placeholder="负责人" readonly></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="联系电话" prop="phone">
                  <el-input v-model="dataForm.phone" placeholder="联系电话" readonly></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="开店地址" prop="address">
                  <el-input v-model="dataForm.address" placeholder="预开店地址" readonly></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="资金成本" prop="money">
                  <el-input v-model="dataForm.money" placeholder="资金成本" readonly></el-input>
                </el-form-item>
              </el-col>
            </el-row>
          </el-collapse-item>
          <el-collapse-item title="申请结果" name="2">
              <el-row>
                  <el-col :span="12">
                      <el-form-item label="实地考察结果" prop="addressResult">
                        <el-input v-model="dataForm.addressResult" placeholder="还未实地考察,请等待..." readonly></el-input>
                      </el-form-item>
                  </el-col>
                  <el-col :span="12">
                      <el-form-item label="管理员投票结果" prop="tpResult">
                        <el-input v-model="dataForm.tpResult" placeholder="还未投票,请等待..." readonly></el-input>
                      </el-form-item>
                  </el-col>
                  <el-col :span="12">
                      <el-form-item label="最终审核结果" prop="finalResult">
                        <el-input v-model="dataForm.finalResult" placeholder="还未最终审核,请等待..." readonly></el-input>
                      </el-form-item>
                  </el-col>
                  <el-col :span="12">
                      <el-form-item label="审核时间" prop="finalTime">
                        <el-input v-model="dataForm.finalTime" placeholder="还未最终审核,请等待..." readonly></el-input>
                      </el-form-item>
                  </el-col>
                  <el-col :span="24">
                      <el-form-item label="当前状态" prop="zztmc">
                        <el-input v-model="dataForm.zztmc" placeholder="当前状态" readonly></el-input>
                      </el-form-item>
                  </el-col>
              </el-row>
          </el-collapse-item>
          <el-collapse-item title="文件上传" name="3">
            <el-table
            :data="dataList"
            border
            v-loading="dataListLoading"
            ref="multipleTable"
            style="width: 100%;border-bottom:1px solid rgba(30, 23, 32, 0.38);border-top:1px solid rgba(30, 23, 32, 0.38);border-left:1px solid rgba(30, 23, 32, 0.38);">
            <el-table-column
              prop="wjlx"
              header-align="center"
              align="center"
              label="文件类型">
            </el-table-column>
            <el-table-column
              prop="wjmc"
              header-align="center"
              align="center"
              label="文件名称"
              width="280">
            </el-table-column>
            <el-table-column
              prop="ztmc"
              header-align="center"
              align="center"
              label="状态">
              <template slot-scope="scope">
                  <span  v-if="scope.row.ztmc=='未上传'" style="color:red">{{ scope.row.ztmc }}</span>
                  <span v-else style="color: green">{{ scope.row.ztmc }}</span>
              </template>
            </el-table-column>
          <el-table-column
              fixed="right"
              header-align="center"
              align="center"
              width="150"
              label="操作">
              <template slot-scope="scope">
                <el-button type="text" size="small" @click="downloadHandle(scope.row.wjdz,scope.row.wjmc)" :disabled="pdhandle(scope.row.ztmc)">下载</el-button>
              </template>
            </el-table-column>
          </el-table>
          </el-collapse-item>
        </el-collapse>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary"  @click="visible = false">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        fjuploadVisible: false,
        visible: false,
        activeNames: ['1', '2', '3'],
        dataListLoading: false,
        dataList: [],
        dataForm: {
          id: 0,
          name: '',
          phone: '',
          address: '',
          money: '',
          yyxk: '',
          yyxkztdm: '',
          wsxk: '',
          wsxkztdm: '',
          addressResult: '',
          grxd: '',
          grxdztdm: '',
          tpResult: '',
          finalResult: '',
          finalTime: '',
          zztdm: '',
          zztmc: ''
        },
        uid: 0,
        dataRule: {
          name: [
            { required: true, message: '负责人不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '联系电话不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '预开店地址不能为空', trigger: 'blur' }
          ],
          money: [
            { required: true, message: '资金成本不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    activated () {
      this.pdhandle()
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/shopjoin_manage/shopjoin/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.shopjoin.name
                this.dataForm.phone = data.shopjoin.phone
                this.dataForm.address = data.shopjoin.address
                this.dataForm.money = data.shopjoin.money
                this.dataForm.yyxk = data.shopjoin.yyxk
                this.dataForm.yyxkztdm = data.shopjoin.yyxkztdm
                this.dataForm.wsxk = data.shopjoin.wsxk
                this.dataForm.wsxkztdm = data.shopjoin.wsxkztdm
                this.dataForm.addressResult = data.shopjoin.addressResult
                this.dataForm.grxd = data.shopjoin.grxd
                this.dataForm.grxdztdm = data.shopjoin.grxdztdm
                this.dataForm.tpResult = data.shopjoin.tpResult
                this.dataForm.finalResult = data.shopjoin.finalResult
                this.dataForm.finalTime = data.shopjoin.finalTime
                this.dataForm.zztdm = data.shopjoin.zztdm
                this.dataForm.zztmc = data.shopjoin.zztmc
                this.uid = data.shopjoin.uid
                this.dataListLoading = true
                this.$http({
                  url: this.$http.adornUrl('/shopjoin_manage/joinfile/detaillist'),
                  method: 'get',
                  params: this.$http.adornParams({
                    key: this.uid
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
              }
            })
          }
        })
      },
      // 数据刷新
      reloaddata () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/shopjoin_manage/joinfile/sxwjlist'),
          method: 'get',
          params: this.$http.adornParams({
            page: this.pageIndex,
            limit: this.pageSize,
            key: this.uid
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
          } else {
            this.dataList = []
          }
          this.dataListLoading = false
        })
      },
      // 判断当前上传状态
      pdhandle (ztmc) {
        if (ztmc === '未上传') {
          return true
        } else {
          return false
        }
      },
      downloadHandle (wjdz, wjmc) {
        var mm = wjdz.split('/')
        this.main = mm[ mm.length - 1 ]
        var ll = wjmc
        this.$http({
          url: this.$http.adornUrl(`/shopjoin_manage/joinfile/downloads/${this.main}?token=${this.$cookie.get('token')}`),
          method: 'post',
          responseType: 'arraybuffer'
        }).then(({data}) => {
          if (data) {
            console.log('文件下载成功')
            let blob = new Blob([data], {
              // word文档为msword,pdf文档为pdf
              type: `application/pdf;charset-UTF-8`
            })
            if ('download' in document.createElement('a')) {
              let objectUrl = URL.createObjectURL(blob)
              console.log(objectUrl)
              let downEle = document.createElement('a')
              let fname = ll
              // 下载文件的名字
              downEle.href = objectUrl
              downEle.setAttribute('download', fname)
              document.body.appendChild(downEle)
              downEle.click()
            } else {
              navigator.msSaveBlob(blob, ll)
            }
          }
        })
      }
    }
  }
</script>
