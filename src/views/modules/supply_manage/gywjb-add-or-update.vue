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
              <el-form-item label="发布人" prop="name">
                <el-input v-model="dataForm.name" placeholder="发布人"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="手机号" prop="phone">
                <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="地址" prop="address">
                <el-input v-model="dataForm.address" placeholder="地址"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="材料名" prop="materialName">
                <el-input v-model="dataForm.materialName" placeholder="材料名"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="材料数量" prop="materialNum">
                <el-input v-model="dataForm.materialNum" placeholder="材料数量"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="材料价格" prop="materialPrice">
                <el-input v-model="dataForm.materialPrice" placeholder="材料价格"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="状态代码" prop="ztdm" v-if="false">
                <el-input v-model="dataForm.ztdm" placeholder="状态代码"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="状态名称" prop="ztmc" v-if="false">
                <el-input v-model="dataForm.ztmc" placeholder="状态名称"></el-input>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="开始日期" prop="ksri">
                 <el-date-picker v-model="dataForm.ksri" type="date" placeholder="请选择" value-format="yyyyMMdd"></el-date-picker>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="结束日期" prop="jsrq">
                 <el-date-picker v-model="dataForm.jsrq" type="date" placeholder="请选择" value-format="yyyyMMdd"></el-date-picker>
              </el-form-item>
          </el-col>
          <el-col :span="12">
              <el-form-item label="送货方式" prop="shfs">
                <el-input v-model="dataForm.shfs" placeholder="送货方式"></el-input>
             </el-form-item>
         </el-col>
        </el-row>
     </el-collapse-item>

     <el-collapse-item title="简介" name="2">
        <el-input v-model="dataForm.intro" placeholder="请输入内容" type="textarea" :rows="9" style="width:100%;" maxlength="2000"  @input="descInput()"> </el-input>  
          <!-- 显示剩余字数 -->
        <span style="float:right; position:absolute; left:580px; top:700px;">剩余： {{this.remnant}}字</span>
      </el-collapse-item>

     <el-collapse-item title="相关文件上传" name="3">
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
    <span>{{this.dataList}}</span>
     </el-collapse-item>
     </el-collapse>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
  <Fjupload v-if="fjuploadVisible" ref="fjupload" @tellFatherName="reloaddata" ></Fjupload>
 </div>
</template>

<script>
  import Fjupload from './uploadform'
  export default {
    data () {
      return {
        fjuploadVisible: false,
        visible: false,
        activeNames: ['1', '2', '3'],
        remnant: 2000,
        dataListLoading: false,
        dataForm: {
          id: 0,
          name: '',
          phone: '',
          address: '',
          materialName: '',
          materialNum: '',
          materialPrice: '',
          intro: '',
          ztdm: '',
          ztmc: '',
          ksri: '',
          jsrq: '',
          wjdz: '',
          wjmc: '',
          shfs: '',
          wjlxmc: '',
          wjlxdm: '',
          zztdm: '',
          zztmc: ''
        },
        uid: 0,
        dataList: [],
        dataRule: {
          name: [
            { required: true, message: '发布人不能为空', trigger: 'blur' }
          ],
          phone: [
            { required: true, message: '手机号不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '地址不能为空', trigger: 'blur' }
          ],
          materialName: [
            { required: true, message: '材料名不能为空', trigger: 'blur' }
          ],
          materialNum: [
            { required: true, message: '材料数量不能为空', trigger: 'blur' }
          ],
          materialPrice: [
            { required: true, message: '材料价格不能为空', trigger: 'blur' }
          ],
          ksri: [
            { required: true, message: '开始日期不能为空', trigger: 'blur' }
          ],
          jsrq: [
            { required: true, message: '结束日期不能为空', trigger: 'blur' }
          ],
          shfs: [
            { required: true, message: '送货方式不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    components: {
      Fjupload
    },
    activated () {
      this.pduploadhandle()
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
              url: this.$http.adornUrl(`/supply_manage/gywjb/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.gywjb.name
                this.dataForm.phone = data.gywjb.phone
                this.dataForm.address = data.gywjb.address
                this.dataForm.materialName = data.gywjb.materialName
                this.dataForm.materialNum = data.gywjb.materialNum
                this.dataForm.materialPrice = data.gywjb.materialPrice
                this.dataForm.intro = data.gywjb.intro
                this.dataForm.ztdm = data.gywjb.ztdm
                this.dataForm.ztmc = data.gywjb.ztmc
                this.dataForm.ksri = data.gywjb.ksri
                this.dataForm.jsrq = data.gywjb.jsrq
                this.dataForm.wjdz = data.gywjb.wjdz
                this.dataForm.wjmc = data.gywjb.wjmc
                this.dataForm.shfs = data.gywjb.shfs
                this.dataForm.wjlxmc = data.gywjb.wjlxmc
                this.dataForm.wjlxdm = data.gywjb.wjlxdm
                this.dataForm.zztdm = data.gywjb.zztdm
                this.dataForm.zztmc = data.gywjb.zztmc
              }
            })
          }
          this.dataListLoading = true
          this.$http({
            url: this.$http.adornUrl('/supply_manage/wjlxb/fbwjlist'),
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
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/supply_manage/gywjb/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'phone': this.dataForm.phone,
                'address': this.dataForm.address,
                'materialName': this.dataForm.materialName,
                'materialNum': this.dataForm.materialNum,
                'materialPrice': this.dataForm.materialPrice,
                'intro': this.dataForm.intro,
                'ksri': this.dataForm.ksri,
                'jsrq': this.dataForm.jsrq,
                'shfs': this.dataForm.shfs,
                'uid': this.uid,
                'zztdm': 1,
                'zztmc': '选购'
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
      },
      // 显示剩余字数
      descInput () {
        var txtVal = this.dataForm.intro.length
        this.remnant = 2000 - txtVal
      },
      // 判断当前上传状态
      pduploadhandle (ztmc) {
        if (ztmc === '未上传') {
          return false
        } else {
          return true
        }
      },
      // 判断当前上传状态
      pdhandle (ztmc) {
        if (ztmc === '未上传') {
          return true
        } else {
          return false
        }
      },
      // 附件上传
      fjuploadHandle (wjlx, wjlxdm, uid) {
        this.uid = uid
        this.fjuploadVisible = true
        this.$nextTick(() => {
          this.$refs.fjupload.initwj(wjlx, wjlxdm, uid)
        })
      },
      downloadHandle (wjdz, wjmc) {
        var mm = wjdz.split('/')
        this.main = mm[ mm.length - 1 ]
        var ll = wjmc
        // this.url2 = this.$http.adornUrl(`/gc/gcscgyxxb/downloads/${this.dataForm.id}?token=${this.$cookie.get('token')}`)
        this.$http({
          url: this.$http.adornUrl(`/supply_manage/gywjb/downloads/${this.main}?token=${this.$cookie.get('token')}`),
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
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对该文件进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
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
                  this.reloaddata()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      // 数据刷新
      reloaddata () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/supply_manage/wjlxb/scwjlist'),
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