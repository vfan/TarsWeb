<template>
  <div class="page_operation_deploy">
    <el-form
      :rules="rules"
      size="small"
      class="operation_deploy_form"
      ref="formvue"
      :model="model"
      inline
      label-position="top"
      itemWidth="480px"
      v-show="deployShow"
      @submit.native.prevent="save"
    >
      <el-row>
        <el-col :span="12">
          <el-form-item prop="application" :label="$t('deployService.form.app')">
            <el-tooltip class="item" effect="dark" :content="$t('deployService.form.appAdd')" placement="top-start">
              <el-autocomplete
                size="small"
                class="inline-input"
                v-model="model.application"
                :fetch-suggestions="queryApplicationLists"
                @select="handleSelect"
              ></el-autocomplete>
            </el-tooltip>
          </el-form-item>
        </el-col>

        <el-col :span="12">
          <el-form-item prop="server_name" :label="$t('deployService.form.serviceName')">
            <el-input
              size="small"
              v-model="model.server_name"
              :placeholder="$t('deployService.form.serviceFormatTips')"
              :required-tip="$t('deployService.form.serviceTips')"
              pattern="^[a-zA-Z]([a-zA-Z0-9]+)?$"
              :pattern-tip="$t('deployService.form.serviceFormatTips')"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item prop="server_type" :label="$t('deployService.form.serviceType')" required>
            <el-select size="small" v-model="model.server_type" required :required-tip="$t('deployService.form.serviceTypeTips')">
              <el-option v-for="d in types" :key="d" :value="d">{{ d }}</el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item prop="template_name" :label="$t('deployService.form.template')" required>
            <el-select size="small" v-model="model.template_name" required :required-tip="$t('deployService.form.templateTips')">
              <el-option v-for="d in templates" :key="d" :value="d">{{ d }}</el-option>
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item prop="node_name" :label="$t('serverList.table.th.ip')">
            <el-select v-model="model.node_name" size="small" filterable>
              <el-option v-for="d in model.nodeList" :key="d" :value="d">
                {{ d }}
              </el-option>
            </el-select>

            <!-- <el-input
          size="small"
          v-model="model.node_name"
          :placeholder="$t('serverList.table.th.ip')"
          required
          :required-tip="$t('deployService.form.nodeTips')"
        ></el-input> -->
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="SET">
            <SetInputer
              :enabled.sync="model.enable_set"
              :name.sync="model.set_name"
              :area.sync="model.set_area"
              :group.sync="model.set_group"
            ></SetInputer>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item :label="$t('user.op')" v-show="enableAuth">
            <el-input size="small" v-model="model.operator" :placeholder="$t('user.tips.sep')"></el-input> </el-form-item
        ></el-col>
        <el-col :span="12">
          <el-form-item :label="$t('user.dev')" v-show="enableAuth">
            <el-input size="small" v-model="model.developer" :placeholder="$t('user.tips.sep')"></el-input> </el-form-item
        ></el-col>
      </el-row>

      <el-table :data="model.adapters">
        <el-table-column label="OBJ">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input
              size="small"
              v-model="props.row.obj_name"
              :placeholder="$t('deployService.form.placeholder')"
              required
              :required-tip="$t('deployService.form.objTips')"
              pattern="^[a-zA-Z0-9]+$"
              :pattern-tip="$t('deployService.form.placeholder')"
            ></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.table.th.endpoint')" width="140px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input size="small" v-model="props.row.bind_ip" placeholder="IP" required :required-tip="$t('deployService.table.tips.ip')"></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.table.th.port')" width="100px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input
              size="small"
              type="number"
              :min="0"
              :max="65535"
              v-model="props.row.port"
              placeholder="0-65535"
              required
              :required-tip="$t('deployService.table.tips.empty')"
            ></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.form.portType')" width="150px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-radio v-model="props.row.port_type" label="tcp">TCP</el-radio>
            <el-radio v-model="props.row.port_type" label="udp">UDP</el-radio>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.table.th.protocol')" width="180px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-radio v-model="props.row.protocol" label="tars">TARS</el-radio>
            <el-radio v-model="props.row.protocol" label="not_tars">{{ $t('serverList.servant.notTARS') }}</el-radio>
          </template>
        </el-table-column>
        <el-table-column :label="$t('deployService.table.th.threads')" width="100px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input
              size="small"
              type="number"
              :min="0"
              v-model="props.row.thread_num"
              required
              :required-tip="$t('deployService.table.tips.empty')"
            ></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('serverList.table.servant.maxConnecttions')" width="120px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input
              size="small"
              type="number"
              :min="0"
              v-model="props.row.max_connections"
              required
              :required-tip="$t('deployService.table.tips.empty')"
            ></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('serverList.table.servant.maxQueue')" width="100px">
          <template slot="header" slot-scope="props">
            <span class="required">{{ props.column.label }}</span>
          </template>
          <template slot-scope="props">
            <el-input
              size="small"
              type="number"
              :min="0"
              v-model="props.row.queuecap"
              required
              :required-tip="$t('deployService.table.tips.empty')"
            ></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('serverList.table.servant.timeout')" width="110px">
          <template slot-scope="props">
            <el-input size="small" type="number" :min="0" v-model="props.row.queuetimeout"></el-input>
          </template>
        </el-table-column>
        <el-table-column :label="$t('operate.operates')" width="100px">
          <template slot-scope="props">
            <el-link @click="addAdapter(props.row)" type="primary">{{ $t('operate.add') }}</el-link>
            <el-link @click="model.adapters.splice(props.$index, 1)" v-if="props.$index" type="danger">{{ $t('operate.delete') }}</el-link>
          </template>
        </el-table-column>
      </el-table>

      <el-button type="button" :loading="isLoadingPort" theme="sub-primary" @click="getAutoPort()">{{ $t('deployService.form.getPort') }}</el-button>
      &nbsp;&nbsp;
      <el-button type="primary" :loading="isSubmiting" @click="save" theme="primary">{{ $t('common.submit') }}</el-button>
    </el-form>

    <let-modal
      v-model="resultModal.show"
      width="700px"
      class="more-cmd"
      :footShow="false"
      @close="closeResultModal"
      style="text-align: center;"
      @on-cancel="closeResultModal"
    >
      <p class="result-text">{{ $t('deployService.form.ret.success') }}{{ $t('resource.installRstMsg') }}</p>
      <let-table :data="resultModal.resultList" :empty-msg="$t('common.nodata')" :row-class-name="resultModal.rowClassName">
        <let-table-column title="ip" prop="ip"> </let-table-column>
        <let-table-column :title="$t('resource.installResult')" prop="rst">
          <template slot-scope="scope">
            <p v-text="scope.row.rst ? $t('common.success') : $t('common.error')"></p>
          </template>
        </let-table-column>
        <let-table-column :title="$t('common.message')" prop="msg"></let-table-column>
      </let-table>
    </let-modal>

    <div style="width: 400px; margin: 0 auto;" v-show="deployModal.show">
      <let-form ref="deployForm" itemWidth="400px">
        <let-form-item :label="$t('nodes.node_name')" required>
          <let-input
            v-model="deployModal.node_name"
            :placeholder="$t('nodes.nodeNameTips')"
            required
            :required-tip="$t('nodes.nodeNameTips')"
          ></let-input>
        </let-form-item>

        <let-form-item :label="$t('pub.dlg.releaseVersion')">
          <let-select v-model="deployModal.model.patch_id" required :required-tip="$t('pub.dlg.ab')">
            <let-option v-for="d in deployModal.model.patchList" :key="d.id" :value="d.id">
              {{ d.id }} | {{ d.posttime }} | {{ d.comment }}
            </let-option>
          </let-select>
        </let-form-item>
      </let-form>
      <div style="width: 100%; text-align: center;">
        <let-tag>{{ $t('deployLog.info') }}</let-tag>
        <let-button type="submit" theme="primary" style="margin: 20px auto;" @click="doDeployLog">{{ $t('deployLog.install') }}</let-button>
      </div>
    </div>

    <PublishStatus ref="publishStatus"></PublishStatus>
  </div>
</template>

<script>
import { Loading } from 'element-ui'

import PublishStatus from '../publish/status'

import SetInputer from '@/components/set-inputer'

const types = ['tars_cpp', 'tars_java', 'tars_php', 'tars_nodejs', 'not_tars', 'tars_go']

const tars_templates = [
  'tars.tarsregistry',
  'tars.tarsAdminRegistry',
  'tars.tarspatch',
  'tars.tarsnode',
  'tars.tarsconfig',
  'tars.tarsnotify',
  'tars.tarsstat',
  'tars.tarsquerystat',
  'tars.tarsproperty',
  'tars.tarsqueryproperty',
  'tars.framework-db',
  'tars.tarslog'
]

const getInitialModel = () => ({
  applicationList: [],
  nodeList: [],
  application: '',
  server_name: '',
  server_type: types[0],
  template_name: '',
  node_name: '',
  enable_set: false,
  set_name: '',
  set_area: '',
  set_group: '',
  operator: '',
  developer: '',
  adapters: [
    {
      obj_name: '',
      bind_ip: '',
      port: '',
      port_type: 'tcp',
      protocol: 'tars',
      thread_num: 5,
      max_connections: 100000,
      queuecap: 50000,
      queuetimeout: 20000
    }
  ]
})

export default {
  name: 'OperationDeploy',
  components: {
    SetInputer,
    PublishStatus
  },

  data() {
    return {
      types,
      tars_templates,
      all_templates: [],
      notars_templates: [],
      templates: [],
      model: getInitialModel(),
      enableAuth: false,
      deployShow: false,
      deployModal: {
        show: false,
        // close: true,
        node_name: '',
        model: {
          patch_id: '',
          patchList: [],
          serverList: []
        }
      },
      resultModal: {
        show: false,
        resultList: [],
        rowClassName: rowData => {
          if (rowData && rowData.row && !rowData.row.rst) {
            return 'err-row'
          }
          return ''
        }
      },
      isLoadingPort: false,
      isSubmiting: false,
      rules: {
        application: [{ required: true, message: this.$t('deployService.form.appTips'), trigger: 'chnage' }],
        server_name: [
          { required: true, message: this.$t('deployService.form.serviceTips'), trigger: 'blur' },
          {
            validator: (rule, value, callback) => {
              if (/^[a-zA-Z]([a-zA-Z0-9]+)?$/.test(value) === false) callback(this.$t('deployService.form.serviceFormatTips'))
              return callback()
            },
            trigger: 'blur'
          }
        ],
        node_name: [{ required: true, message: this.$t('deployService.form.nodeTips'), trigger: 'blur' }]
      }
    }
  },
  mounted() {
    this.$ajax
      .getJSON('/server/api/is_enable_auth')
      .then(data => {
        this.enableAuth = data.enableAuth || false
      })
      .catch(err => {})

    this.$ajax
      .getJSON('/server/api/application_list')
      .then(data => {
        this.model.applicationList = data
      })
      .catch(err => {
        this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
      })

    this.$ajax
      .getJSON('/server/api/node_list')
      .then(data => {
        console.log(data)
        this.model.nodeList = data
      })
      .catch(err => {
        this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
      })

    this.$ajax
      .getJSON('/server/api/template_name_list')
      .then(data => {
        this.templates = data
        this.all_templates = data
        this.notars_templates = []

        this.all_templates.forEach(e => {
          var index
          for (index = 0; index < this.tars_templates.length; index++) {
            if (this.tars_templates[index] == e) return
          }
          this.notars_templates.push(e)
        })

        this.model.template_name = data[0]

        // var application = document.querySelector('#inputApplication .let-select__filter__input')
        // var that = this

        // application.onblur = function () {
        //   that.changeApplication(this.value)
        // }
      })
      .catch(err => {
        this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
      })
    this.$watch('model.application', (val, oldVal) => {
      if (val === oldVal) {
        return
      }
      this.changeApplication(val)
    })

    this.$watch('model.node_name', (val, oldVal) => {
      if (val === oldVal) {
        return
      }

      this.model.adapters.forEach(d => {
        d.bind_ip = val // eslint-disable-line no-param-reassign
      })
    })

    this.checkDeployLog()
  },

  methods: {
    handleSelect(item) {
      console.log(item)
    },
    queryApplicationLists(queryString, cb) {
      var applicationList = this.model.applicationList
      var results = queryString ? applicationList.filter(this.createFilter(queryString)) : applicationList
      // 调用 callback 返回建议列表的数据
      results = results.map(item => {
        return { value: item, label: item }
      })
      cb(results)
    },
    createFilter(queryString) {
      return restaurant => {
        return restaurant.toLowerCase().indexOf(queryString.toLowerCase()) === 0
      }
    },
    changeApplication(app) {
      if (app == 'tars') {
        this.templates = this.tars_templates
      } else {
        this.templates = this.notars_templates
      }
    },
    addAdapter(template) {
      this.model.adapters.push(Object.assign({}, template))
    },
    deploy() {
      this.$confirm(this.$t('deployService.form.deployServiceTip'), this.$t('common.alert')).then(() => {
        // const loading = this.$Loading.show()
        const loadingDeploy = Loading.service({ fullscreen: true })

        this.$ajax
          .postJSON('/server/api/deploy_server', this.model)
          .then(data => {
            // loading.hide()
            this.$nextTick(() => {
              loadingDeploy.close()
            })
            if (data.tars_node_rst && data.tars_node_rst.length) {
              this.showResultModal(data.tars_node_rst)
            } else {
              this.$message.success(this.$t('deployService.form.ret.success'))
            }
            this.model = getInitialModel()
            this.model.template_name = this.templates[0]
          })
          .catch(err => {
            // loading.hide()
            this.$nextTick(() => {
              loadingDeploy.close()
            })

            this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
          })
      })
    },
    getAutoPort() {
      // const loading = this.$Loading.show()

      this.isLoadingPort = true
      // const loading = Loading.service()

      var adapters = this.model.adapters
      var bindIps = []
      adapters.forEach(adapter => {
        bindIps.push(adapter.bind_ip)
      })
      this.$ajax
        .getJSON('/server/api/auto_port', { node_name: bindIps.join(';') })
        .then(data => {
          // loading.close()
          this.isLoadingPort = false
          data.forEach((node, index) => {
            console.log('index', index, node)
            this.$set(adapters[index], 'port', String(node.port || ''))
          })
        })
        .catch(err => {
          // loading.close()
          this.isLoadingPort = false
          this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
        })
    },
    save() {
      console.log('save', this.model)

      // return
      // var application = document.querySelector('#inputApplication .let-select__filter__input').value.trim()

      // if (this.model.application == '') {
      //   // this.model.application = application
      //   return
      // }

      // this.$message.error(this.$t('deployService.form.nameTips'))
      this.$refs.formvue.validate((valid, validObjs) => {
        if (!valid) {
          return false
        }
        const model = this.model
        this.isSubmiting = true

        // const loading = this.$Loading.show()
        this.$ajax
          .getJSON('/server/api/server_exist', {
            application: model.application,
            server_name: model.server_name,
            node_name: model.server_name
          })
          .then(isExists => {
            // loading.hide()
            this.isSubmiting = false
            if (isExists) {
              this.$message.error(this.$t('deployService.form.nameTips'))
            } else {
              this.deploy()
            }
          })
          .catch(err => {
            this.isSubmiting = false
            this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
          })
      })
      // if (this.$refs.form.validate()) {

      // }
    },

    showResultModal(data) {
      this.resultModal.resultList = data
      this.resultModal.show = true
    },

    closeResultModal() {
      this.resultModal.show = false
      this.resultModal.resultList = []
    },
    checkDeployLog() {
      this.$ajax
        .getJSON('/server/api/need_deploy_log')
        .then(data => {
          this.deployModal.show = data.need
          this.deployShow = !data.need
          if (data.need) {
            this.showDeployLog()
          }
        })
        .catch(err => {
          this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
        })
    },
    getPatchList(application, serverName, currPage, pageSize) {
      return this.$ajax.getJSON('/server/api/server_patch_list', {
        application,
        module_name: serverName,
        curr_page: currPage,
        page_size: pageSize
      })
    },
    showDeployLog() {
      const application = 'tars'
      const server_name = 'tarslog'
      this.deployModal.model = {
        application: application,
        server_name: server_name,
        patch_id: '',
        patchList: []
      }
      this.getPatchList(application, server_name, 1, 10).then(data => {
        this.deployModal.model.patchList = data.rows
      })
    },
    doDeployLog() {
      if (this.$refs.deployForm.validate()) {
        this.$ajax
          .getJSON('/server/api/expand_deploy_log', { node_name: this.deployModal.node_name })
          .then(data => {
            this.deployModal.model.serverList = data.server

            this.$refs.publishStatus.savePublishServer(this.deployModal, this.onCancel)
          })
          .catch(err => {
            this.$message.error(`${this.$t('deployLog.failed')}: ${err.err_msg || err.message}`)
          })
      }
    },
    onCancel() {
      this.checkDeployLog()
    }
  }
}
</script>

<style lang="postcss">
.operation_deploy_form .el-row {
  width: 1030px;
}
.operation_deploy_form .el-row .el-input {
  width: 480px;
}
.page_operation_deploy {
  .let-table {
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
