<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon>
      </el-alert>
      <el-row class="cate-opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader v-model="selectedCateKeys" :options="cateList" :props="cateCascaderProps"
            @change="handleCascaderChange" clearable>
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tabs标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleTabsClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddParamsDialog">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" style="width: 100%" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleTagClose(index, scope.row)">
                  {{item}}
                </el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible === true" v-model="scope.row.inputValue" ref="saveTagInputRef" size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#">
            </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini"
                  @click="showEditParamsDialog(scope.row.attr_id)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddParamsDialog">添加属性</el-button>
          <!-- 静态参数表格 -->
          <el-table :data="onlyTableData" style="width: 100%" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, index) in scope.row.attr_vals" :key="index" closable @close="handleTagClose(index, scope.row)">
                  {{item}}
                </el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputVisible === true" v-model="scope.row.inputValue" ref="saveTagInputRef" size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#">
            </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini"
                  @click="showEditParamsDialog(scope.row.attr_id)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteParams(scope.row.attr_id)">删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog :title="'添加' + tabsText" :visible.sync="isAddParamsDialogVisible" width="40%"
      @closed="addParamsDialogClosed">
      <el-form :model="addParamsForm" :rules="addParamsFormRules" ref="addParamsFormRef" label-width="80px">
        <el-form-item :label="tabsText" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isAddParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog :title="'修改' + tabsText" :visible.sync="isEditParamsDialogVisible" width="40%"
      @closed="editParamsDialogClosed">
      <el-form :model="editParamsForm" :rules="editParamsFormRules" ref="editParamsFormRef" label-width="80px">
        <el-form-item :label="tabsText" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isEditParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      cateList: [],
      cateCascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedCateKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      isAddParamsDialogVisible: false,
      addParamsForm: {
        attr_name: ''
      },
      addParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入属性名称', trigger: 'blur' }
        ]
      },
      isEditParamsDialogVisible: false,
      editParamsForm: {
        attr_name: ''
      },
      editParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入属性名称', trigger: 'blur' }
        ]
      }
    }
  },
  created: function () {
    this.getCateList()
  },
  methods: {
    getCateList: async function () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateList = res.data
    },
    handleCascaderChange: function () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      this.getCateParamsList()
    },
    handleTabsClick: function () {
      if (this.selectedCateKeys.length !== 3) return
      this.getCateParamsList()
    },
    getCateParamsList: async function () {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

      res.data.forEach(item => {
        if (item.attr_vals) {
          item.attr_vals = item.attr_vals.split(' ')
        } else {
          item.attr_vals = []
        }
        item.inputVisible = false
        item.inputValue = ''
      })

      if (this.activeName === 'many') {
        this.manyTableData = res.data
        console.log(this.manyTableData)
      } else {
        this.onlyTableData = res.data
      }
    },
    showAddParamsDialog: function () {
      this.isAddParamsDialogVisible = true
    },
    addParamsDialogClosed: function () {
      this.$refs.addParamsFormRef.resetFields()
    },
    addParams: function () {
      this.$refs.addParamsFormRef.validate(async isValid => {
        if (!isValid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getCateParamsList()
        this.isAddParamsDialogVisible = false
      })
    },
    showEditParamsDialog: async function (id) {
      this.isEditParamsDialogVisible = true
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
        params: {
          attr_sel: this.activeName
        }
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.editParamsForm = res.data
    },
    editParams: function () {
      this.$refs.editParamsFormRef.validate(async isValid => {
        if (!isValid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`, {
          attr_name: this.editParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getCateParamsList()
        this.isEditParamsDialogVisible = false
      })
    },
    editParamsDialogClosed: function () {
      this.$refs.editParamsFormRef.resetFields()
    },
    deleteParams: async function (id) {
      const confirm = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(cancel => cancel)
      if (confirm !== 'confirm') return this.$message.info('已经取消删除')
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getCateParamsList()
    },
    handleInputConfirm: function (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue)
      this.saveAttrValues(row)
      row.inputValue = ''
      row.inputVisible = false
    },
    saveAttrValues: async function (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
    },
    showInput: function (row) {
      row.inputVisible = true

      this.$nextTick(() => {
        this.$refs.saveTagInputRef.focus()
      })
      console.log(row)
    },
    handleTagClose: function (index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrValues(row)
    }

  },
  computed: {
    isBtnDisabled: function () {
      if (this.selectedCateKeys.length !== 3) return true
      return false
    },
    cateId: function () {
      if (this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
      return null
    },
    tabsText: function () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cate-opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 100px;
}
</style>
