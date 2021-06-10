<template>
  <div>
    <el-dialog
      :title="!dataForm.id ? '新增' : '修改'"
      :close-on-click-modal="false"
      :visible.sync="visible">
      <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-collapse v-model="activeNames">
          <el-collapse-item title="基本信息" name="1">
            <el-row>
              <el-col :span="12">
                <el-form-item label="负责人" prop="name">
                  <el-input v-model="dataForm.name" placeholder="负责人"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="联系电话" prop="phone">
                  <el-input v-model="dataForm.phone" placeholder="联系电话"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="开店地址" prop="address">
                  <el-input v-model="dataForm.address" placeholder="预开店地址"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="资金成本" prop="money">
                  <el-input v-model="dataForm.money" placeholder="资金成本"></el-input>
                </el-form-item>
              </el-col>
            </el-row>
          </el-collapse-item>
          <el-collapse-item title="文件上传" name="2">
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
                <el-button type="text" size="small" @click="fjuploadHandle(scope.row.wjlx, scope.row.wjlxdm, scope.row.uid)" :disabled="pduploadhandle(scope.row.ztmc)">上传</el-button>
                <el-button type="text" size="small" @click="deleteHandle(scope.row.id)" :disabled="pdhandle(scope.row.ztmc)">删除</el-button>
                <el-button type="text" size="small" @click="downloadHandle(scope.row.wjdz,scope.row.wjmc)" :disabled="pdhandle(scope.row.ztmc)">下载</el-button>
              </template>
            </el-table-column>
          </el-table>
            <!--<el-form-item label="营业许可" prop="yyxk">
              <el-input v-model="dataForm.yyxk" placeholder="营业许可"></el-input>
            </el-form-item>
            <el-form-item label="营业许可状态代码" prop="yyxkztdm">
              <el-input v-model="dataForm.yyxkztdm" placeholder="营业许可状态代码"></el-input>
            </el-form-item>
            <el-form-item label="卫生许可" prop="wsxk">
              <el-input v-model="dataForm.wsxk" placeholder="卫生许可"></el-input>
            </el-form-item>
            <el-form-item label="卫生许可状态代码" prop="wsxkztdm">
              <el-input v-model="dataForm.wsxkztdm" placeholder="卫生许可状态代码"></el-input>
            </el-form-item>-->
          </el-collapse-item>
          {{this.dataList}}
        </el-collapse>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </span>
    </el-dialog>
    <Joinupload v-if="fjuploadVisible" ref="joinupload" @tellFatherName="reloaddata" ></Joinupload>
  </div>
</template>

<script>
  import Joinupload from './joinuploadform'
  export default {
    data () {
      return {
        fjuploadVisible: false,
        visible: false,
        activeNames: ['1', '2'],
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
    components: {
      Joinupload
    },
    activated () {
      this.pduploadhandle()
      this.pdhandle()
    },
    methods: {
      init (id) {
        console.log(id)
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
                this.dataListLoading = true
                this.$http({
                  url: this.$http.adornUrl('/shopjoin_manage/joinfile/xgwjlist'),
                  method: 'get',
                  params: this.$http.adornParams({
                    key: this.dataForm.id
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
          } else {
            this.dataListLoading = true
            this.$http({
              url: this.$http.adornUrl('/shopjoin_manage/joinfile/sqwjlist'),
              method: 'get'
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
      },
      // 附件上传
      fjuploadHandle (wjlx, wjlxdm, uid) {
        this.uid = uid
        this.fjuploadVisible = true
        this.$nextTick(() => {
          this.$refs.joinupload.initwj(wjlx, wjlxdm, uid)
        })
      },
       // 判断当前上传状态
      pduploadhandle (ztmc) {
        if (ztmc === '未上传') {
          return false
        } else {
          return true
        }
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
      // 删除
      deleteHandle (id) {
        this.$confirm(`确定对该文件进行删除操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/shopjoin_manage/joinfile/sqdelete'),
            method: 'post',
            data: this.$http.adornData(id, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.reloaddata()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
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
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/shopjoin_manage/shopjoin/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'money': this.dataForm.money,
                'uid': this.uid
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
