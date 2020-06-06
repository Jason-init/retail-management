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
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="roleList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, index1) in scope.row.children" :key="item1.id"
              :class="['border-bottom', index1 === 0 ? 'border-top' : '', 'vertical-center']">
              <el-col :span="5">
                <el-tag closable @close='tagClosed(scope.row,item1.id)'>{{item1.authName}}</el-tag><i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2, index2) in item1.children" :key="item2.id" :class="[index2 === 0 ? '' : 'border-top', 'vertical-center']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close='tagClosed(scope.row,item2.id)'>{{item2.authName}}</el-tag><i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="(item3) in item2.children" :key="item3.id" type="warning" closable @close='tagClosed(scope.row,item3.id)'>{{item3.authName}}</el-tag>
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
          <template>
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      roleList: []
    }
  },
  created: function () {
    this.getRoleList()
  },
  methods: {
    getRoleList: async function () {
      const { data: res } = await this.$http.get('roles')
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
      const { data: res } = await this.$http.delete(`roles/${row.id}/rights/${id}}`)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      row.children = res.data
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
