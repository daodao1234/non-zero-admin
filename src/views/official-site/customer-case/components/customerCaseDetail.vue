<template>
  <div class="createCustomerCase-container">
    <el-form ref="customerCaseForm" :model="customerCaseForm" :rules="rules" class="form-container">

      <sticky :class-name="'sub-navbar '+customerCaseForm.isEnable">
        <el-button v-loading="loading" style="margin-left: 10px;" type="success" @click="submitForm">发布
        </el-button>
        <el-button v-loading="loading" type="warning" @click="draftForm">草稿</el-button>
      </sticky>

      <div class="createCustomerCase-main-container">
        <el-row>

          <el-col :span="24">
            <el-form-item style="margin-bottom: 40px;" prop="title">
              <MDinput v-model="customerCaseForm.firstHeading" :maxlength="100" name="name" required>
                标题
              </MDinput>
            </el-form-item>

            <el-row>
              <el-col :span="12" :xs="12" :sm="24" :lg="12">
                <el-form-item style="margin-bottom: 40px;" label-width="80px" label="二级标题:">
                  <el-input :rows="1" v-model="customerCaseForm.secondaryHeading" type="textarea" class="article-textarea" autosize placeholder="请输入内容"/>
                </el-form-item>
              </el-col>
              <el-col :span="12" :xs="12" :sm="24" :lg="12">
                <el-form-item style="margin-bottom: 40px;" label-width="80px" label="三级标题:">
                  <el-input :rows="1" v-model="customerCaseForm.tertiaryHeading" type="textarea" class="article-textarea" autosize placeholder="请输入内容"/>
                </el-form-item>
              </el-col>
            </el-row>

            <div class="customerCaseInfo-container">
              <el-row>
                <el-col :span="12" :xs="12" :sm="24" :lg="12">
                  <el-form-item label-width="80px" label="类别:" class="customerInfo-container-item">
                    <el-select v-model="customerCaseForm.caseClass" placeholder="请选择">
                      <el-option
                        v-for="item in customerCaseClassOptions"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"/>
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col :span="12" :xs="12" :sm="24" :lg="12" >
                  <el-form-item label-width="80px" label="创建时间:" class="postInfo-container-item">
                    <el-date-picker v-model="customerCaseForm.createdTime" type="datetime" format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期时间"/>
                  </el-form-item>
                </el-col>
              </el-row>
            </div>
          </el-col>
        </el-row>

        <el-form-item style="margin-bottom: 40px;" label-width="80px" label="摘要:">
          <el-input :rows="1" v-model="customerCaseForm.customerCaseSynopsis" type="textarea" class="article-textarea" autosize placeholder="请输入内容"/>
          <span v-show="synopsisLength" class="word-counter">{{ synopsisLength }}字</span>
        </el-form-item>

        <el-form-item style="margin-bottom: 40px;" label-width="80px" label="图片:">
          <el-upload
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :on-success="handleSuccess"
            :headers="myHeaders"
            :file-list="fileList"
            :action="uploadUrl()"
            list-type="picture-card">
            <i class="el-icon-plus"/>
          </el-upload>
          <el-dialog :visible.sync="picVisible">
            <img :src="customerCaseForm.casePic" width="100%" alt="">
          </el-dialog>
        </el-form-item>

        <div class="editor-container">
          <Tinymce ref="editor" :height="400" v-model="customerCaseForm.caseText" />
        </div>
      </div>
    </el-form>

  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'
import MDinput from '@/components/MDinput'
import Sticky from '@/components/Sticky' // 粘性header组件
import { getToken } from '@/utils/auth'
import { fetchClazzList } from '@/api/official-site/base-info/clazzConf'
import { fetchCustomerCase, createCustomerCase, updateCustomerCase } from '@/api/official-site/customer-case/customer-case'

const defaultForm = {
  id: undefined,
  createdTime: undefined,
  firstHeading: '',
  secondaryHeading: '',
  tertiaryHeading: '',
  customerCaseSynopsis: '',
  caseText: '',
  casePic: '',
  caseClass: '',
  isEnable: 'draft'
}

export default {
  name: 'CustomerCaseDetail',
  components: { Tinymce, MDinput, Sticky },
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value === '') {
        this.$message({
          message: rule.field + '为必传项',
          type: 'error'
        })
        callback(new Error(rule.field + '为必传项'))
      } else {
        callback()
      }
    }
    return {
      customerCaseForm: Object.assign({}, defaultForm),
      loading: false,
      customerCaseClassOptions: [], // 从数据库获取类别
      myHeaders: {
        'x-auth-token': getToken() // 文件上传携带token
      },
      editFlag: this.isEdit,
      fileList: [],
      picVisible: false,
      rules: {
        first_heading: [{ validator: validateRequire }],
        case_text: [{ validator: validateRequire }],
        case_pic: [{ validator: validateRequire }]
      }
    }
  },
  computed: {
    synopsisLength() {
      return this.customerCaseForm.customerCaseSynopsis.length
    }
  },
  created() {
    fetchClazzList({ clazzName: 'CASE' }).then(response => {
      if (response.data.code === 20000) {
        this.customerCaseClassOptions = response.data.data
      }
    })
    if (this.isEdit) {
      const id = this.$route.params && this.$route.params.id
      this.fetchData(id)
    } else {
      this.customerCaseForm = Object.assign({}, defaultForm)
    }
  },
  methods: {
    uploadUrl() {
      return process.env.BASE_API + '/official/website/customer/case/upload'
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log(file)
      this.customerCaseForm.casePic = file.response.data
      this.picVisible = true
    },
    handleSuccess(res, file, fileList) {
      if (res.code === 20000) {
        this.customerCaseForm.casePic = res.data
      }
    },
    fetchData(id) {
      fetchCustomerCase(id).then(response => {
        this.customerCaseForm = response.data.data
        this.fileList.splice(0, this.fileList.length) // 清空
        this.fileList.push({ name: response.data.data.id, url: response.data.data.casePic })
      }).catch(err => {
        console.log(err)
      })
    },
    submitForm() {
      console.log(this.customerCaseForm)
      this.$refs.customerCaseForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.customerCaseForm.isEnable = '1' // published
          if (!this.editFlag) {
            createCustomerCase(this.customerCaseForm).then(response => {
              if (response.data.code === 20000) {
                this.customerCaseForm.id = response.data.data.id
                this.editFlag = true // 第一次点击发布后,isEdit标志修改为false,避免后续点击发布按钮重新添加为新的文章
                this.$notify({
                  title: '成功',
                  message: '发布案例成功',
                  type: 'success',
                  duration: 2000
                })
              } else {
                this.$notify({
                  title: '失败',
                  message: '发布案例失败',
                  type: 'fail',
                  duration: 2000
                })
              }
            })
          } else {
            updateCustomerCase(this.customerCaseForm).then(response => {
              if (response.data.code === 20000) {
                this.$notify({
                  title: '成功',
                  message: '更新案例成功',
                  type: 'success',
                  duration: 2000
                })
              } else {
                this.$notify({
                  title: '失败',
                  message: '更新案例失败',
                  type: 'fail',
                  duration: 2000
                })
              }
            })
          }
          this.loading = false
        } else {
          console.log('error submit!!')
          this.$message.error('信息填写有错误')
          return false
        }
      })
    },
    draftForm() {
      if (this.customerCaseForm.caseText.length === 0 || this.customerCaseForm.firstHeading.length === 0) {
        this.$message({
          message: '请填写必要的标题和内容',
          type: 'warning'
        })
        return
      }
      this.loading = true
      this.customerCaseForm.isEnable = '0' // draft
      if (!this.editFlag) {
        createCustomerCase(this.customerCaseForm).then(response => {
          this.customerCaseForm.id = response.data.data.id
          this.editFlag = true // 第一次点击发布后,isEdit标志修改为false,避免后续点击发布按钮重新添加为新的文章
          if (response.data.code === 20000) {
            this.$notify({
              title: '成功',
              message: '保存案例草稿成功',
              type: 'success',
              duration: 2000
            })
          } else {
            this.$notify({
              title: '失败',
              message: '保存案例草稿失败',
              type: 'fail',
              duration: 2000
            })
          }
        })
      } else {
        updateCustomerCase(this.customerCaseForm).then(response => {
          if (response.data.code === 20000) {
            this.$notify({
              title: '成功',
              message: '更新案例草稿成功',
              type: 'success',
              duration: 2000
            })
          } else {
            this.$notify({
              title: '失败',
              message: '更新案例草稿失败',
              type: 'fail',
              duration: 2000
            })
          }
        })
      }
      this.loading = false
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "src/styles/mixin.scss";
  .createCustomerCase-container {
    position: relative;
    .createCustomerCase-main-container {
      padding: 40px 45px 20px 50px;
      .customerCaseInfo-container {
        position: relative;
        @include clearfix;
        margin-bottom: 10px;
        .customerCaseInfo-container-item {
          float: left;
        }
      }
      .editor-container {
        min-height: 500px;
        margin: 0 0 30px;
        .editor-upload-btn-container {
          text-align: right;
          margin-right: 10px;
          .editor-upload-btn {
            display: inline-block;
          }
        }
      }
    }
    .word-counter {
      width: 40px;
      position: absolute;
      right: -10px;
      top: 0px;
    }
  }
</style>
