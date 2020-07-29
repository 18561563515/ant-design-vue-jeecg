<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form
        :form="form"
        slot="detail"
      >
        <a-row>
          <a-col :span="24">
            <a-form-item
              label="姓名"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <a-input
                v-decorator="['name', validatorRules.name]"
                placeholder="请输入姓名"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="性别"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <j-dict-select-tag
                type="radio"
                v-decorator="['sex', validatorRules.sex]"
                :trigger-change="true"
                dict-code="sex"
                placeholder="请选择性别"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="生日"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <j-date
                placeholder="请选择生日"
                v-decorator="['birthday']"
                :trigger-change="true"
                style="width: 100%"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="账户余额"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <a-input-number
                v-decorator="['account']"
                placeholder="请输入账户余额"
                style="width: 100%"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="个人描述"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <j-editor v-decorator="['futext',{trigger:'input'}]" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="账户状态"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <j-dict-select-tag
                type="list"
                v-decorator="['statu']"
                :trigger-change="true"
                dict-code="zhuangtai"
                placeholder="请选择账户状态"
                v-if="!disabled"
              />
              <div v-else>
                <a-tag color="pink">
                  审核未通过
                </a-tag>
                <a-tag color="blue">
                  审核通过
                </a-tag>
                <a-tag color="purple">
                  等待审核
                </a-tag>
              </div>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              label="个人爱好"
              :label-col="labelCol"
              :wrapper-col="wrapperCol"
            >
              <j-multi-select-tag
                type="checkbox"
                v-decorator="['hobby']"
                :trigger-change="true"
                dict-code="aihao"
                placeholder="请选择个人爱好"
              />
            </a-form-item>
          </a-col>
          <a-col
            v-if="showFlowSubmitButton"
            :span="24"
            style="text-align: center"
          >
            <a-button @click="submitForm">
              提 交
            </a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>
import {
  httpAction,
  getAction
} from '@/api/manage'
import pick from 'lodash.pick'
import {
  validateDuplicateValue
} from '@/utils/util'
import JFormContainer from '@/components/jeecg/JFormContainer'
import JDate from '@/components/jeecg/JDate'
import JDictSelectTag from '@/components/dict/JDictSelectTag'
import JMultiSelectTag from '@/components/dict/JMultiSelectTag'
import JEditor from '@/components/jeecg/JEditor'

export default {
  name: 'CesDemoForm',
  components: {
    JFormContainer,
    JDate,
    JDictSelectTag,
    JMultiSelectTag,
    JEditor
  },
  props: {
    // 标签
    chakan: {
      type: Number,
      default: 0
    },
    // 流程表单data
    formData: {
      type: Object,
      default: () => {},
      required: false
    },
    // 表单模式：false流程表单 true普通表单
    normal: {
      type: Boolean,
      default: false,
      required: false
    },
    // 表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data () {
    return {
      form: this.$form.createForm(this),
      model: {},
      labelCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 5
        }
      },
      wrapperCol: {
        xs: {
          span: 24
        },
        sm: {
          span: 16
        }
      },
      confirmLoading: false,
      validatorRules: {
        name: {
          rules: [{
            required: true,
            message: '请输入姓名!'
          }]
        },
        sex: {
          rules: [{
            required: true,
            message: '请输入性别!'
          }]
        }
      },
      url: {
        add: '/customer/cesDemo/add',
        edit: '/customer/cesDemo/edit',
        queryById: '/customer/cesDemo/queryById'
      }
    }
  },
  computed: {
    formDisabled () {
      if (this.normal === false) {
        if (this.formData.disabled === false) {
          return false
        } else {
          return true
        }
      }
      return this.disabled
    },
    showFlowSubmitButton () {
      if (this.normal === false) {
        if (this.formData.disabled === false) {
          return true
        } else {
          return false
        }
      } else {
        return false
      }
    }
  },
  created () {
    // 如果是流程中表单，则需要加载流程表单data
    this.showFlowData()
  },
  mounted () {
    setTimeout(() => {
      console.log(this.form)
    }, 2000)
  },
  methods: {
    add () {
      this.edit({})
    },
    edit (record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'name', 'sex', 'birthday', 'account', 'futext', 'statu', 'hobby'))
      })
    },
    // 渲染流程表单数据
    showFlowData () {
      if (this.normal === false) {
        let params = {
          id: this.formData.dataId
        }
        getAction(this.url.queryById, params).then((res) => {
          if (res.success) {
            this.edit(res.result)
          }
        })
      }
    },
    submitForm () {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }
          let formData = Object.assign(this.model, values)
          console.log('表单提交数据', formData)
          httpAction(httpurl, formData, method).then((res) => {
            if (res.success) {
              that.$message.success(res.message)
              that.$emit('ok')
            } else {
              that.$message.warning(res.message)
            }
          }).finally(() => {
            that.confirmLoading = false
          })
        }
      })
    },
    popupCallback (row) {
      this.form.setFieldsValue(pick(row, 'name', 'sex', 'birthday', 'account', 'futext', 'statu', 'hobby'))
    }

  }
}
</script>
