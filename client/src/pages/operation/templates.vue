<template>
  <div class="page_operation_templates">
    <el-button size="small" theme="primary" style="float: right;" @click="addItem">{{ $t('template.btn.addTempate') }}</el-button>
    <el-form inline itemWidth="200px">
      <el-form-item :label="$t('deployService.form.template')">
        <el-input size="small" v-model="query.template_name"></el-input>
      </el-form-item>
      <el-form-item :label="$t('template.search.parentTemplate')">
        <el-input size="small" v-model="query.parents_name"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button :loading="isSearching" size="small" type="primary" @click="search" theme="primary">{{ $t('operate.search') }}</el-button>
      </el-form-item>
    </el-form>

    <el-table v-loading="isSearching" ref="table" :data="items" :empty-text="$t('common.nodata')">
      <el-table-column :label="$t('deployService.form.template')" prop="template_name" width="250"></el-table-column>
      <el-table-column :label="$t('template.search.parentTemplate')" prop="parents_name" width="250"></el-table-column>
      <el-table-column :label="$t('cfg.btn.lastUpdate')" prop="posttime"></el-table-column>
      <el-table-column :label="$t('operate.operates')" width="300px">
        <template slot-scope="scope">
          <el-link type="primary" @click="mergeItem(scope.row)">{{ $t('operate.merge') }}</el-link>
          <el-link type="primary" @click="viewItem(scope.row)">{{ $t('operate.view') }}</el-link>
          <el-link type="primary" @click="editItem(scope.row)">{{ $t('operate.update') }}</el-link>
          <el-link type="primary" @click="removeItem(scope.row)">{{ $t('operate.delete') }}</el-link>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :visible.sync="viewModal.show" :title="$t('template.view.title')" width="800px">
      <pre v-if="viewModal.model">{{ viewModal.model.profile }}</pre>
      <div slot="foot"></div>
    </el-dialog>

    <el-dialog
      :visible.sync="detailModal.show"
      :title="detailModal.isNew ? this.$t('template.add.title') : this.$t('template.update.title')"
      width="800px"
      @on-confirm="saveItem"
      @on-cancel="closeDetailModal"
    >
      <el-form ref="detailForm" :model="detailModal.model" v-if="detailModal.model" itemWidth="700px">
        <el-form-item
          prop="template_name"
          :label="$t('deployService.form.template')"
          :rules="[
            {
              required: true,
              message: $t('template.add.templateNameTips')
            }
          ]"
        >
          <el-input
            size="small"
            v-model="detailModal.model.template_name"
            :placeholder="$t('template.add.templateFormatTips')"
            required
            :required-tip="$t('template.add.templateNameTips')"
            pattern="^[a-zA-Z]([.a-zA-Z0-9]+)?$"
            :pattern-tip="$t('template.add.templateFormatTips')"
          ></el-input>
        </el-form-item>
        <el-form-item
          :rules="[
            {
              required: true,
              message: $t('deployService.table.tips.empty')
            }
          ]"
          prop="parents_name"
          :label="$t('template.search.parentTemplate')"
          required
        >
          <el-select
            size="small"
            v-model="detailModal.model.parents_name"
            :placeholder="$t('pub.dlg.defaultValue')"
            required
            :required-tip="$t('deployService.table.tips.empty')"
          >
            <el-option value>{{ $t('pub.dlg.defaultValue') }}</el-option>
            <el-option v-for="d in items" :key="d.id" :value="d.template_name">{{ d.template_name }}</el-option>
          </el-select>
        </el-form-item>
        <el-form-item
          :rules="[
            {
              required: true,
              message: $t('deployService.table.tips.empty')
            }
          ]"
          prop="profile"
          :label="$t('template.form.content')"
        >
          <el-input
            type="textarea"
            :rows="12"
            size="small"
            v-model="detailModal.model.profile"
            required
            :required-tip="$t('deployService.table.tips.empty')"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button :loading="isSaveItemLoading" size="small" type="primary" @click="saveItem">{{ $t('common.submit') }}</el-button>
          <el-button size="small" @click="closeDetailModal">{{ $t('common.cancel') }}</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'OperationTemplates',

  data() {
    return {
      isSearching: false,
      isSaveItemLoading: false,
      query: {
        template_name: '',
        parents_name: ''
      },
      items: [],
      viewModal: {
        show: false,
        model: null
      },
      detailModal: {
        show: false,
        model: null,
        isNew: false
      }
    }
  },

  mounted() {
    this.fetchData()
  },

  methods: {
    fetchData() {
      // const loading = this.$refs.table.$loading.show()
      this.isSearching = true
      return this.$ajax
        .getJSON('/server/api/query_profile_template', this.query)
        .then(data => {
          this.isSearching = false
          // loading.hide()
          this.items = data
        })
        .catch(err => {
          this.isSearching = false
          // loading.hide()
          this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
        })
    },

    search() {
      this.fetchData()
    },

    closeDetailModal() {
      this.$refs.detailForm.resetFields()
      this.detailModal.show = false
      this.detailModal.model = null
    },

    addItem() {
      this.detailModal.model = {}
      this.detailModal.show = true
      this.detailModal.isNew = true
    },

    viewItem(d) {
      this.viewModal.model = d
      this.viewModal.show = true

      // this.$alert(`<pre>${d.profile}</pre>`, this.$t('template.view.title'))
    },

    editItem(d) {
      this.detailModal.model = d
      this.detailModal.show = true
      this.detailModal.isNew = false
    },
    mergeItem(model) {
      // const loading = this.$Loading.show()
      this.$ajax
        .getJSON('/server/api/get_merge_profile_template', { template_name: model.template_name })
        .then(data => {
          // loading.hide()
          console.log(data)
          model.profile = data.template
          this.viewModal.model = model
          this.viewModal.show = true
        })
        .catch(err => {
          // loading.hide()
          this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
        })
    },
    saveItem() {
      this.$refs.detailForm.validate((valid, validObj) => {
        this.isSaveItemLoading = true
        const model = this.detailModal.model
        const url = model.id ? '/server/api/update_profile_template' : '/server/api/add_profile_template'

        // const loading = this.$Loading.show()
        this.$ajax
          .postJSON(url, model)
          .then(() => {
            // loading.hide()
            this.isSaveItemLoading = false
            this.$message.success(this.$t('common.success'))
            this.closeDetailModal()
            this.fetchData()
          })
          .catch(err => {
            // loading.hide()
            this.isSaveItemLoading = false
            this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
          })
      })
    },

    removeItem(d) {
      this.$confirm(this.$t('template.delete.confirmTips'), this.$t('common.alert'))
        .then(() => {
          // const loading = this.$Loading.show()
          this.$ajax
            .getJSON('/server/api/delete_profile_template', { id: d.id })
            .then(() => {
              // loading.hide()
              this.fetchData().then(() => {
                this.$message.success(this.$t('common.success'))
              })
            })
            .catch(err => {
              // loading.hide()
              this.$message.error(`${this.$t('common.error')}: ${err.message || err.err_msg}`)
            })
        })
        .catch(() => {})
    }
  }
}
</script>

<style>
.page_operation_templates {
  pre {
    color: #909fa3;
    margin-top: 32px;
  }

  .let_modal__body {
    overflow-y: visible;
  }
}
</style>
