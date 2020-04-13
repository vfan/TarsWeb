<template>
  <div class="page_operation_expand">
    <!-- 预扩容配置 -->

    <el-form :model="model" ref="configFormVue" inline label-position="top" itemWidth="480px" @submit.native.prevent="previewExpand">
      <el-form-item
        prop="application"
        :label="$t('deployService.form.app')"
        itemWidth="240px"
        :rules="[{ required: true, message: $t('deployService.table.tips.empty') }]"
      >
        <el-select
          style="width: 240px;"
          size="small"
          v-model="model.application"
          required
          :required-tip="$t('deployService.table.tips.empty')"
          @change="changeSelect('application')"
        >
          <el-option v-for="d in applications" :key="d" :value="d">
            {{ d }}
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item
        :label="$t('serverList.table.th.service')"
        itemWidth="240px"
        prop="server_name"
        :rules="[{ required: true, message: $t('deployService.table.tips.empty') }]"
      >
        <el-select
          style="width: 240px;"
          size="small"
          v-model="model.server_name"
          required
          :required-tip="$t('deployService.table.tips.empty')"
          @change="changeSelect('server_name')"
        >
          <el-option v-for="d in serverNames" :key="d" :value="d">{{ d }}</el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="Set" itemWidth="240px" prop="set" :rules="[{ required: true, message: $t('deployService.table.tips.empty') }]">
        <el-select
          style="width: 240px;"
          size="small"
          v-model="model.set"
          :required-tip="$t('deployService.table.tips.empty')"
          @change="changeSelect('set')"
        >
          <el-option v-for="d in sets" :key="d" :value="d ? d : -1" :label="d ? d : $t('serviceExpand.form.disableSet')"> </el-option>
        </el-select>
      </el-form-item>
      <el-form-item
        :label="$t('serverList.table.th.ip')"
        itemWidth="240px"
        prop="node_name"
        :rules="[{ required: true, message: $t('deployService.table.tips.empty') }]"
      >
        <el-select style="width: 240px;" size="small" v-model="model.node_name" required :required-tip="$t('deployService.table.tips.empty')">
          <el-option v-for="d in nodeNames" :key="d" :value="d">{{ d }}</el-option>
        </el-select>
      </el-form-item>
      <div>
        <el-form-item
          :label="$t('serviceExpand.form.tarIP')"
          itemWidth="100%"
          prop="expandIpStr"
          :rules="[{ required: true, message: $t('deployService.table.tips.empty') }]"
        >
          <el-input
            type="textarea"
            :rows="3"
            style="width: 980px;"
            v-model="model.expandIpStr"
            :placeholder="$t('serviceExpand.form.placeholder')"
            required
            :required-tip="$t('deployService.table.tips.empty')"
          >
          </el-input>
        </el-form-item>
      </div>

      <el-form-item :label="$t('serverList.table.th.enableSet')">
        <SetInputer
          :enabled.sync="model.enable_set"
          :name.sync="model.set_name"
          :area.sync="model.set_area"
          :group.sync="model.set_group"
        ></SetInputer>
      </el-form-item>
      <div>
        <el-form-item :label="$t('serviceExpand.form.nodeConfig')" itemWidth="100%">
          <el-checkbox v-model="model.copy_node_config">
            {{ $t('serviceExpand.form.copyNodeConfig') }}
          </el-checkbox>
        </el-form-item>
      </div>
      <el-button @click="previewExpand" type="primary" theme="primary">{{ $t('serviceExpand.form.preExpand') }}</el-button>
    </el-form>

    <!-- 预扩容列表 -->
    <el-form ref="expandForm" inline class="mt40" v-show="previewItems.length > 0">
      <el-table @selection-change="previewExpandItemsSelectChange" ref="table" :data="previewItems" :empty-text="$t('common.nodata')">
        <el-table-column type="selection" width="55"> </el-table-column>

        <el-table-column :label="$t('historyList.dlg.th.c2')" prop="application"></el-table-column>
        <el-table-column :label="$t('historyList.dlg.th.c3')" prop="server_name"></el-table-column>
        <el-table-column title="Set" prop="set"></el-table-column>
        <el-table-column :label="$t('serverList.servant.objName')" prop="obj_name"></el-table-column>
        <el-table-column :label="$t('historyList.dlg.th.c4')" prop="node_name"></el-table-column>
        <el-table-column :label="$t('deployService.table.th.endpoint')">
          <template slot-scope="scope">
            <el-input v-model="scope.row.bind_ip"></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.table.th.port')">
          <template slot-scope="scope">
            <el-input v-model="scope.row.port"></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.form.template')" prop="template_name"></el-table-column>
        <el-table-column :label="$t('historyList.dlg.th.c8')" prop="status"></el-table-column>
      </el-table>

      <el-button type="button" theme="sub-primary" @click="getAutoPort">{{ $t('deployService.form.getPort') }} </el-button>
      <el-button :loading="isExpanding" type="primary" theme="primary" @click="expand">{{ $t('deployService.title.expand') }}</el-button>
    </el-form>

    <el-dialog v-model="resultModal.show" width="700px" class="more-cmd" :footShow="false" @close="closeResultModal" @on-cancel="closeResultModal">
      <p class="result-text">{{ $t('serviceExpand.form.errTips.success') }}{{ $t('resource.installRstMsg') }}</p>
      <el-table :data="resultModal.resultList" :empty-msg="$t('common.nodata')" :row-class-name="resultModal.rowClassName">
        <el-table-column label="ip" prop="ip"> </el-table-column>
        <el-table-column :label="$t('resource.installResult')" prop="rst">
          <template slot-scope="scope">
            <p v-text="scope.row.rst ? $t('common.success') : $t('common.error')"></p>
          </template>
        </el-table-column>
        <el-table-column :label="$t('common.message')" prop="msg"></el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SetInputer from '@/components/set-inputer'

const ipReg = '((\\d{1,2}|1\\d\\d|2[0-4]\\d|25[0-5])\\.){3}(\\d{1,2}|1\\d\\d|2[0-4]\\d|25[0-5])'
const getInitialModel = () => ({
  application: '',
  server_name: '',
  set: '',
  node_name: '',
  expandIpStr: '',

  expand_nodes: [],
  enable_set: false,
  set_name: '',
  set_area: '',
  set_group: '',
  copy_node_config: false
})

export default {
  name: 'OperationExpand',

  components: {
    SetInputer
  },

  data() {
    return {
      isExpanding: false,
      model: getInitialModel(),
      applications: [],
      serverNames: [],
      sets: [],
      nodeNames: [],
      previewItems: [],
      selectedPreviewItems: [],
      ipReg: `^${ipReg}$`,
      isCheckedAll: false,
      resultModal: {
        show: false,
        resultList: [],
        rowClassName: rowData => {
          if (rowData && rowData.row && !rowData.row.rst) {
            return 'err-row'
          }
          return ''
        }
      }
    }
  },

  mounted() {
    this.getCascadeSelectServer(
      {
        level: 1
      },
      this.$t('common.error')
    ).then(data => {
      this.applications = data
    })
  },

  methods: {
    previewExpandItemsSelectChange(val) {
      this.selectedPreviewItems = val
    },
    changeSelect(attr) {
      switch (attr) {
        case 'application':
          // 修改应用
          this.model.server_name = ''
          this.serverNames = []
          if (this.model.application) {
            this.getCascadeSelectServer(
              {
                level: 2,
                application: this.model.application
              },
              this.$t('common.error')
            ).then(data => {
              this.serverNames = data
            })
          }
          break
        case 'server_name':
          // 修改服务
          this.model.set = ''
          this.sets = []
          if (this.model.server_name) {
            this.getCascadeSelectServer(
              {
                level: 3,
                application: this.model.application,
                server_name: this.model.server_name
              },
              this.$t('common.error')
            ).then(data => {
              this.sets = data
            })
          }
          break
        case 'set': // 修改set
          this.model.node_name = ''
          this.model.nodeName = []
          if (this.model.set) {
            const modelSet = parseInt(this.model.set, 10) === -1 ? '' : this.model.set
            this.getCascadeSelectServer(
              {
                level: 4,
                application: this.model.application,
                server_name: this.model.server_name,
                set: modelSet
              },
              this.$t('common.error')
            ).then(data => {
              this.nodeNames = data
            })
          }
          break
        default:
          break
      }
    },
    getCascadeSelectServer(params, prefix = this.$t('common.error')) {
      return this.$ajax
        .getJSON('/server/api/cascade_select_server', params)
        .then(data => data)
        .catch(err => {
          this.$message.error(`${prefix}: ${err.message || err.err_msg}`)
        })
    },
    indexOf(array, val) {
      for (var i = 0; i < array.length; i++) {
        if (array[i] == val) return i
      }
      return -1
    },
    previewExpand() {
      this.$refs.configFormVue.validate((valid, validObj) => {
        if (!valid) {
          return
        }

        const model = Object.assign({}, this.model)
        model.set = parseInt(model.set, 10) === -1 ? '' : model.set
        model.expand_nodes = model.expandIpStr.trim().split(/[,;\n]/)

        const index = this.indexOf(model.expand_nodes, this.model.node_name)
        if (index != -1) {
          this.$message.error(`${this.$t('serviceExpand.exists')}`)
          return
        }

        // 把空的都去掉
        model.expand_nodes = model.expand_nodes.filter(item => item.trim() != '')
        // const loading = this.$Loading.show()
        this.$ajax
          .postJSON('/server/api/expand_server_preview', model)
          .then(data => {
            // loading.hide()
            const items = data || []
            items.forEach(item => {
              item.isChecked = false
            })
            // hw-todo: el-ui bug
            this.isCheckedAll = false
            this.previewItems = items
          })
          .catch(err => {
            // loading.hide()
            this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
          })
      })
      // 预扩容
      // if (this.$refs.configForm.validate()) {
      // }
    },
    getAutoPort() {
      // var adapters = this.previewItems.filter(item => item.status === this.$t('serviceExpand.form.noExpand') && item.isChecked)
      const adapters = this.selectedPreviewItems.filter(item => item.status === this.$t('serviceExpand.form.noExpand'))
      if (adapters.length == 0) {
        this.$message.error(this.$t('serviceExpand.form.errTips.nodeCheck'))
        return
      }

      // const loading = this.$Loading.show()

      var bindIps = []
      adapters.forEach(adapter => {
        bindIps.push(adapter.bind_ip)
      })

      this.$ajax
        .getJSON('/server/api/auto_port', { node_name: bindIps.join(';') })
        .then(data => {
          // loading.hide()
          data.forEach((node, index) => {
            this.$set(adapters[index], 'port', String(node.port || ''))
          })
        })
        .catch(err => {
          // loading.hide()
          this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
        })
    },
    expand() {
      // 扩容
      // if (this.$refs.expandForm.validate()) {
      // const previewItems = this.previewItems.filter(item => item.status === this.$t('serviceExpand.form.noExpand') && item.isChecked)
      const previewItems = this.selectedPreviewItems.filter(item => item.status === this.$t('serviceExpand.form.noExpand'))
      if (previewItems.length > 0) {
        this.isExpanding = true
        const previewServers = []
        previewItems.forEach(item => {
          previewServers.push({
            bind_ip: item.bind_ip,
            node_name: item.node_name,
            obj_name: item.obj_name,
            port: item.port,
            set: item.set
          })
        })
        const params = {
          application: this.model.application,
          server_name: this.model.server_name,
          set: parseInt(this.model.set, 10) === -1 ? '' : this.model.set,
          node_name: this.model.node_name,
          copy_node_config: this.model.copy_node_config,
          expand_preview_servers: previewServers
        }

        // console.log(params);

        // const loading = this.$Loading.show()
        this.$ajax
          .postJSON('/server/api/expand_server', params)
          .then(data => {
            // hw-todo: 是否需要更新状态待定
            /*
              this.previewItems.forEach((item) => {
                if (item.status === '未扩容' && item.isChecked) {
                  item.status = '已存在';
                }
              }); */
            // loading.hide()
            this.isExpanding = false
            if (data.tars_node_rst && data.tars_node_rst.length) {
              this.showResultModal(data.tars_node_rst)
            } else {
              this.$message.success(this.$t('serviceExpand.form.errTips.success'))
            }
          })
          .catch(err => {
            this.isExpanding = false
            // loading.hide()
            this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
          })
      } else {
        this.$message.error(this.$t('serviceExpand.form.errTips.noneNodes'))
      }
      // }
    },

    showResultModal(data) {
      this.resultModal.resultList = data
      this.resultModal.show = true
    },

    closeResultModal() {
      this.resultModal.show = false
      this.resultModal.resultList = []
    }
  },
  watch: {
    isCheckedAll() {
      const isCheckedAll = this.isCheckedAll
      this.previewItems.forEach(item => {
        item.isChecked = isCheckedAll
      })
    }
  }
}
</script>
<style lang="postcss">
.page_operation_expand {
  .mt40 {
    margin-top: 40px;
  }

  tr.err-row td {
    background: #f56c77 !important;
    color: #fff;
  }

  .result-text {
    margin-top: 20px;
    margin-bottom: 10px;
  }
  .el-table {
    margin: 20px 0 36px;

    th span.required {
      display: inline-block;

      &:after {
        color: #f56c6c;
        content: '*';
        margin-left: 4px;
      }
    }

    tr.err-row td {
      background: #f56c77 !important;
      color: #fff;
    }
  }

  .result-text {
    margin-top: 20px;
    margin-bottom: 10px;
  }

  .set_inputer_item {
    float: left;
    margin-right: 8px;
    width: 126px;
  }
}
</style>
