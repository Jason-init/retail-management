<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="isAddRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="roleList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, index1) in scope.row.children" :key="item1.id"
              :class="['border-bottom', index1 === 0 ? 'border-top' : '', 'vertical-center']">
              <el-col :span="5">
                <el-tag closable @close='tagClosed(scope.row,item1.id)'>{{item1.authName}}</el-tag><i
                  class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2, index2) in item1.children" :key="item2.id"
                  :class="[index2 === 0 ? '' : 'border-top', 'vertical-center']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close='tagClosed(scope.row,item2.id)'>{{item2.authName}}</el-tag><i
                      class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable
                      @close='tagClosed(scope.row,item3.id)'>{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>{{scope.row}}</pre> -->
          </template>
        </el-table-column>
        <el-table-column type="index" label="#">
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述">
        </el-table-column>
        <el-table-column label="操作" width="284">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoleDialog(scope.row.id)">编辑
            </el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRoleDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="isAddRoleDialogVisible" width="40%" @closed="addRoleDialogClosed">
      <el-form :model="addRoleForm" :rules="addRoleFormRules" ref="addRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isAddRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户对话框 -->
    <el-dialog title="编辑角色" :visible.sync="isEditRoleDialogVisible" width="40%" @closed="editRoleDialogClosed">
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isEditRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="isSetRoleDialogVisible" width="40%" @closed="setRoleDialogClosed">
      <el-tree ref="rightTreeRef" :data="rightList" :props="rightTreeProps" show-checkbox default-expand-all node-key="id" :default-checked-keys="defaultKeys" check-on-click-node></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isSetRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      roleList: [],
      isAddRoleDialogVisible: false,
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      addRoleFormRules: {
        roleName: [{
          required: true,
          message: '请输入角色名称',
          trigger: 'blur'
        }]
      },
      isEditRoleDialogVisible: false,
      editRoleForm: {},
      editRoleFormRules: {
        roleName: [{
          required: true,
          message: '请输入角色名称',
          trigger: 'blur'
        }]
      },
      isSetRoleDialogVisible: false,
      rightList: [],
      rightTreeProps: {
        children: 'children',
        label: 'authName'
      },
      defaultKeys: [],
      roleId: ''
    }
  },
  created: function () {
    this.getRoleList()
  },
  methods: {
    getRoleList: async function () {
      const {
        data: res
      } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.roleList = res.data
    },
    tagClosed: async function (row, id) {
      const confirm = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(cancel => cancel)
      if (confirm !== 'confirm') return this.$message.info('已取消删除')
      const {
        data: res
      } = await this.$http.delete(`roles/${row.id}/rights/${id}}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      row.children = res.data
    },
    addRoleDialogClosed: function () {
      this.$refs.addRoleFormRef.resetFields()
    },
    addRole: function () {
      this.$refs.addRoleFormRef.validate(async isValidated => {
        if (!isValidated) return
        const {
          data: res
        } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.isAddRoleDialogVisible = false
        this.getRoleList()
      })
    },
    editRoleDialogClosed: function () {
      this.$refs.editRoleFormRef.resetFields()
    },
    showEditRoleDialog: async function (id) {
      const {
        data: res
      } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.editRoleForm = res.data
      this.isEditRoleDialogVisible = true
    },
    editRole: function () {
      this.$refs.editRoleFormRef.validate(async isValidated => {
        if (!isValidated) return
        const {
          data: res
        } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
          roleName: this.editRoleForm.roleName,
          roleDesc: this.editRoleForm.roleDesc
        })
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.isEditRoleDialogVisible = false
        this.getRoleList()
      })
    },
    deleteRole: async function (id) {
      const confirm = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(cancel => cancel)
      if (confirm !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getRoleList()
    },
    showSetRoleDialog: function (row) {
      this.roleId = row.id
      this.getRightList()
      this.getLeafKeys(row, this.defaultKeys)
      this.isSetRoleDialogVisible = true
    },
    getRightList: async function () {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.rightList = res.data
    },
    getLeafKeys: function (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRoleDialogClosed: function () {
      this.defaultKeys = []
    },
    setRole: async function () {
      const keys = [...this.$refs.rightTreeRef.getCheckedKeys(), ...this.$refs.rightTreeRef.getHalfCheckedKeys()].join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: keys })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.isSetRoleDialogVisible = false
      this.getRoleList()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.border-top {
  border-top: 1px solid #eee;
}
.border-bottom {
  border-bottom: 1px solid #eee;
}
.vertical-center {
  display: flex;
  align-items: center;
}
</style>
