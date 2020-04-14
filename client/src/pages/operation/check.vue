<template>
  <div class="page_operation_templates">
    <el-button size="small" type="primary" @click="checkFramework">{{ $t('nodes.btn.check') }}</el-button>
    <el-button size="small" type="info" style="float: right;" @click="openShowProblem">有问题怎么办?</el-button>
    <br /><br />
    <el-table ref="checkLoading" :data="servers" stripe :row-class-name="tableRowClassName" :empty-msg="$t('common.nodata')">
      <el-table-column width="200" :label="$t('checkTable.table.th.server_name')" prop="serverName"></el-table-column>
      <el-table-column :label="$t('checkTable.table.th.node_name')" prop="nodeName"></el-table-column>
      <el-table-column :label="$t('checkTable.table.th.obj_name')" prop="objName"></el-table-column>
      <el-table-column width="200" :label="$t('checkTable.table.th.status')" prop="status"></el-table-column>
    </el-table>
    <el-dialog :visible.sync="showProblem" :title="$t('checkTable.problemDialog.title')" v-model="showProblem" width="600px">
      <div>
        <br />
        <p>{{ $t('checkTable.problemDialog.info') }}</p>
        <br />
        <p>{{ $t('checkTable.problemDialog.restartFramework') }}</p>
        <p>{{ $t('checkTable.problemDialog.stopFramework') }}</p>
        <p>{{ $t('checkTable.problemDialog.restartServer') }}</p>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'OperationCheck',

  data() {
    return {
      showProblem: false,
      servers: []
    }
  },

  mounted() {},

  methods: {
    openShowProblem() {
      this.showProblem = true
    },
    tableRowClassName({ row, rowIndex }) {
      if (row.check === 1) {
        return ''
      } else if (row.check === 2) {
        return 'checking-row'
      } else if (row.check === 0) {
        return 'success-row'
      } else if (row.check === -1) {
        return 'warning-row'
      }
      return ''
    },
    checkFramework() {
      // const loading = this.$refs.checkLoading.$loading.show()
      this.$ajax
        .getJSON('/server/api/get_framework_list')
        .then(data => {
          // loading.hide()
          for (var i = 0; i < data.servers.length; i++) {
            data.servers[i].check = 1
            data.servers[i].status = 'waiting'
          }

          this.servers = data.servers

          var that = this
          for (var i = 0; i < that.servers.length; i++) {
            ;(function (i) {
              setTimeout(function () {
                that.checkServer(i)
              }, i * 100)
            })(i)
          }
        })
        .catch(err => {
          // loading.hide()

          console.error(err)
          this.$notify({
            title: this.$t('checkTable.adminRegistryFailed'),
            message: this.$t('checkTable.restartAdminRegistry'),
            type: 'error',
            duration: 0
          })
        })
    },
    checkServer(index) {
      var server = this.servers[index]

      server.check = 2
      server.status = 'checking'

      this.$ajax
        .postJSON('/server/api/check_framework_server', { server: server })
        .then(data => {
          server.check = 0
          server.status = 'succ'
        })
        .catch(err => {
          console.error('checkserver', err)
          server.check = -1
          server.status = this.$t('checkTable.failed')
        })
    }
  }
}
</script>

<style scoped>
.page_operation_templates {
  pre {
    color: #909fa3;
    margin-top: 32px;
  }
}

.el-table tr.checking-row td {
  background: burlywood !important;
  color: #fff;
}
.el-table tr.warning-row td {
  background: #f56c77 !important;
  color: #fff;
}
.el-table tr.success-row td {
  background: #6accab !important;
  color: #fff;
}
.el-table tr.checking-row td .el-table__operation {
  color: #fff;
}
.el-table tr.warning-row td .el-table__operation {
  color: #fff;
}
.el-table tr.success-row td .el-table__operation {
  color: #fff;
}
</style>
