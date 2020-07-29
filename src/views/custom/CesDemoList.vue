<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form
        layout="inline"
        @keyup.enter.native="searchQuery"
      >
        <a-row :gutter="24">
          <a-col
            :xl="6"
            :lg="7"
            :md="8"
            :sm="24"
          >
            <a-form-item label="姓名">
              <a-input
                placeholder="请输入姓名"
                v-model="queryParam.name"
              />
            </a-form-item>
          </a-col>
          <a-col
            :xl="6"
            :lg="7"
            :md="8"
            :sm="24"
          >
            <a-form-item label="账户状态">
              <j-dict-select-tag
                placeholder="请选择账户状态"
                v-model="queryParam.statu"
                dict-code="zhuangtai"
              />
            </a-form-item>
          </a-col>
          <a-col
            :xl="6"
            :lg="7"
            :md="8"
            :sm="24"
          >
            <span
              style="float: left;overflow: hidden;"
              class="table-page-search-submitButtons"
            >
              <a-button
                type="primary"
                @click="searchQuery"
                icon="search"
              >查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
              <a
                @click="handleToggleSearch"
                style="margin-left: 8px"
              >
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button
        @click="handleAdd"
        type="primary"
        icon="plus"
      >
        新增
      </a-button>
      <a-button
        type="primary"
        icon="download"
        @click="handleExportXls('7-24测试')"
      >
        导出
      </a-button>
      <a-upload
        name="file"
        :show-upload-list="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button
          type="primary"
          icon="import"
        >
          导入
        </a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item
            key="1"
            @click="batchDel"
          >
            <a-icon type="delete" />
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div
        class="ant-alert ant-alert-info"
        style="margin-bottom: 16px;"
      >
        <i class="anticon anticon-info-circle ant-alert-icon" /> 已选择 <a style="font-weight: 600">{{
          selectedRowKeys.length }}</a>项
        <a
          style="margin-left: 24px"
          @click="onClearSelected"
        >清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        row-key="id"
        :columns="columns"
        :data-source="dataSourceA"
        :pagination="ipagination"
        :loading="loading"
        :row-selection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange"
      >
        <template slot="tags">
          <span>
            <a-tag
              :color="i.statu==2?'red':(i.statu==1?'blue':'purple')"
              v-for="(i,index) in dataSource[columns]"
              :key="index"
            >{{ i.statu_dictText }}</a-tag>
            <!-- <a-tag color="red" >未审核通过</a-tag>
          <a-tag color="purple" >等待审核</a-tag> -->
          </span>
        </template>
        <template
          slot="htmlSlot"
          slot-scope="text"
        >
          <div v-html="text" />
        </template>
        <template
          slot="imgSlot"
          slot-scope="text"
        >
          <span
            v-if="!text"
            style="font-size: 12px;font-style: italic;"
          >无图片</span>
          <img
            v-else
            :src="getImgView(text)"
            height="25px"
            alt=""
            style="max-width:80px;font-size: 12px;font-style: italic;"
          >
        </template>
        <template
          slot="fileSlot"
          slot-scope="text"
        >
          <span
            v-if="!text"
            style="font-size: 12px;font-style: italic;"
          >无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)"
          >
            下载
          </a-button>
        </template>

        <span
          slot="action"
          slot-scope="text, record"
        >
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多
              <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail1(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm
                  title="确定删除吗?"
                  @confirm="() => handleDelete(record.id)"
                >
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a @click="tanchu(record)">系统自定义</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>

    <ces-demo-modal
      ref="modalForm"
      @ok="modalFormOk"
      :chakan="chakan"
    />
  </a-card>
</template>

<script>
import {
  deleteAction,
  getAction,
  downFile,
  getFileAccessHttpUrl
} from '@/api/manage'
import '@/assets/less/TableExpand.less'
import {
  mixinDevice
} from '@/utils/mixin'
import {
  JeecgListMixin
} from '@/mixins/JeecgListMixin'
import CesDemoModal from './modules/CesDemoModal'

import JDictSelectTag from '@/components/dict/JDictSelectTag.vue'
import {
  filterMultiDictText
} from '@/components/dict/JDictSelectUtil'

export default {
  name: 'CesDemoList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    JDictSelectTag,
    CesDemoModal
  },
  data () {
    return {
      /* 数据源 */
      dataSourceA: [],
      description: '7-24测试管理页面',
      // 标签状态
      chakan: 0,
      // 表头
      columns: [{
        title: '#',
        dataIndex: '',
        key: 'rowIndex',
        width: 60,
        align: 'center',
        customRender: function (t, r, index) {
          return parseInt(index) + 1
        }
      },
      {
        title: '姓名',
        align: 'center',
        dataIndex: 'name'
      },
      {
        title: '性别',
        align: 'center',
        dataIndex: 'sex_dictText'
      },
      {
        title: '生日',
        align: 'center',
        dataIndex: 'birthday',
        customRender: function (text) {
          return !text ? '' : (text.length > 10 ? text.substr(0, 10) : text)
        }
      },
      {
        title: '账户余额',
        align: 'center',
        dataIndex: 'account'
      },
      {
        title: '个人描述',
        align: 'center',
        dataIndex: 'futext',
        width: 300,
        scopedSlots: {
          customRender: 'htmlSlot'
        }
      },
      {
        title: '账户状态',
        align: 'center',
        dataIndex: 'statu_dictText',
        // scopedSlots: { customRender: 'tags' },
        customRender: (text, record, index) => {
          // console.log(record)
          if (text == '审核通过') {
            return <a-tag color="green">审核通过</a-tag>

          } else if (text == '审核拒绝') {
            return <a-tag color="pink">审核未通过</a-tag>
          } else {
            return <a-tag color="blue">等待审核</a-tag>
          }
        }
      },
      {
        title: '个人爱好',
        align: 'center',
        dataIndex: 'hobby_dictText'
      },
      {
        title: '操作',
        dataIndex: 'action',
        align: 'center',
        fixed: 'right',
        width: 147,
        scopedSlots: {
          customRender: 'action'
        }
      }
      ],
      url: {
        list: '/customer/cesDemo/list',
        delete: '/customer/cesDemo/delete',
        deleteBatch: '/customer/cesDemo/deleteBatch',
        exportXlsUrl: '/customer/cesDemo/exportXls',
        importExcelUrl: 'customer/cesDemo/importExcel'

      },
      dictOptions: {}
    }
  },
  created () {
    console.log('dataSource111111', this.dataSourceA)
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    loadData (arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      // 加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams()// 查询条件
      this.loading = true
      getAction(this.url.list, params).then((res) => {
        if (res.success) {
          this.dataSourceA = res.result.records
          // console.log('dataSource2222', this.dataSource)
          this.ipagination.total = res.result.total
        }
        if (res.code === 510) {
          this.$message.warning(res.message)
        }
        this.loading = false
      })
    },
    initDictConfig () {},
    tanchu (row) {
      console.log(row)
      this.$message.error(row.name)
      // console.log(this.$refs.modalForm)
    },
    handleDetail1: function (record) {
      this.$refs.modalForm.edit(record)
      this.$refs.modalForm.title = '详情'
      this.$refs.modalForm.disableSubmit = true
      this.chakan = 1

    }
  },
  mounted () {
    console.log('dataSource333333', this.dataSourceA)
  }
}
</script>
<style scoped>
	@import '~@assets/less/common.less';
</style>
