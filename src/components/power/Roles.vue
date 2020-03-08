<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <!-- 添加角色按钮区域 -->
        <el-col>
          <el-button @click="showAddRoleDialog" type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row
              :class="['border', 'vCenter']"
              v-for="(item1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag @close="removeRightsById(scope.row, item1.id)" closable>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="['borderT', 'vCenter']"
                  v-for="item2 in item1.children"
                  :key="item2.id"
                >
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      @close="removeRightsById(scope.row, item2.id)"
                      closable
                      type="success"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightsById(scope.row, item3.id)"
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="285px">
          <template v-slot="scope">
            <el-button
              @click="showEditRoleDialog(scope.row)"
              size="mini"
              type="promary"
              icon="el-icon-edit"
            >编辑</el-button>
            <el-button
              size="mini"
              @click="removeRole(scope.row.id)"
              type="danger"
              icon="el-icon-delete"
            >删除</el-button>
            <el-button
              @click="showSetRightsDialog(scope.row)"
              size="mini"
              type="warning"
              icon="el-icon-setting"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog @close="addRoleClosed" title="添加角色" :visible.sync="addRoleVisible" width="30%">
      <!-- 内容主体区域 -->
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色信息对话框 -->
    <el-dialog @close="editRoleClosed" title="编辑角色信息" :visible.sync="editRoleVisible" width="30%">
      <!-- 内容主体区域 -->
      <el-form ref="editFormRef" :rules="addFormRules" :model="editForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightsDialogVisible"
      width="50%">
      <!-- 内容主体区域 -->
      <el-tree
        ref="treeRef"
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        :default-checked-keys="defaultKeys"
      ></el-tree>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      roleList: [],
      // 控制添加角色对话框的显示与隐藏
      addRoleVisible: false,
      // 控制添加角色对话框的显示与隐藏
      editRoleVisible: false,
      // 控制分配角色权限对话框的显示与隐藏
      setRightsDialogVisible: false,
      // 添加角色表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑角色表单数据
      editForm: {},
      // 所有权限的数据
      rightsList: {},
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 角色id
      roleId: null,
      // 默认选中的节点id值数组
      defaultKeys: [],
      // 添加角色表单和编辑角色表单的验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 2, max: 8, message: '角色名长度为2~8个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
          {
            min: 2,
            max: 20,
            message: '角色描述长度为2~20个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getData()
  },
  methods: {
    async getData() {
      const { data: res } = await this.$axios.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.roleList = res.data
    },
    // 点击显示添加角色对话框的事件处理函数
    showAddRoleDialog() {
      this.addRoleVisible = true
    },
    // 监听添加角色对话框关闭事件函数
    addRoleClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加角色
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$axios.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        this.addRoleVisible = false
        this.getData()
      })
    },
    // 点击显示添加角色对话框的事件处理函数
    async showEditRoleDialog(row) {
      const { data: res } = await this.$axios.get('roles/' + row.id)
      if (res.meta.status !== 200) {
        return
      }
      this.editForm = res.data
      this.editRoleVisible = true
    },
    editRoleClosed() {
      this.$refs.editFormRef.clearValidate()
    },
    editRole() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$axios.put(
          'roles/' + this.editForm.roleId,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑角色信息提交失败！')
        }
        this.editRoleVisible = false
        this.getData()
        this.$message.success('编辑成功！')
      })
    },
    // 删除角色
    async removeRole(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return
      }
      const { data: res } = await this.$axios.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！')
      }
      this.editRoleVisible = false
      this.getData()
      this.$message.success('删除成功！')
    },
    // 根据id删除对应的权限
    async removeRightsById(role, rightsId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return
      }
      const { data: res } = await this.$axios.delete(
        `roles/${role.id}/rights/${rightsId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      this.$message.success('删除成功！')
      role.children = res.data
    },
    async showSetRightsDialog(role) {
      const { data: res } = await this.$axios.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限信息失败！')
      }
      this.roleId = role.id
      // 先清空数组
      this.defaultKeys = []
      this.rightsList = res.data
      // 获取三级权限的id，放入 this.defaultKeys数组中
      this.getLeafKeys(role, this.defaultKeys)
      this.setRightsDialogVisible = true
    },
    // 深度优先搜索，获取角色下所有三级权限的id，并保存到 defaultKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const { data: res } = await this.$axios.post(`roles/${this.roleId}/rights`, {
        rids: keys.join(',')
      })
      if (res.meta.status !== 200) { return this.$message.error('分配权限失败！') }
      this.setRightsDialogVisible = false
      this.getData()
      this.$message.success('分配权限成功！')
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px 5px;
}
.border:nth-of-type(1) {
  border-top: 1px solid #eee;
}
.border {
  border-bottom: 1px solid #eee;
}
.borderT:nth-of-type(1) {
  border-top: none;
}
.borderT {
  border-top: 1px solid #eee;
}
.vCenter {
  display: flex;
  align-items: center;
}
</style>
