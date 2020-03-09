<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button @click="showAddCateDialog" type="primary">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        class="treeTable"
        show-index
        :expand-type="false"
        index-text="#"
        :selection-type="false"
        :data="catelist"
        :columns="columns"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template v-slot:isok="scope">
          <i
            class="el-icon-success"
            style="color:lightgreen"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i class="el-icon-error" style="color:red" v-else></i>
        </template>
        <!-- 排序 -->
        <template v-slot:order="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 1" size="mini" type="success">二级</el-tag>
          <el-tag v-else size="mini" type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template v-slot:operation="scope">
          <el-button size="mini" type="primary" @click="showEditCateDialog(scope.row)" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" @click="deleteCate(scope.row)" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[5, 10]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" @close="addCateDialogClosed" :visible.sync="addCateDialogVisible" width="50%">
      <!-- 添加分类的表单 -->
      <el-form
        ref="addCateFormRef"
        :rules="addCateFormRules"
        :model="addCateForm"
        label-width="90px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类 -->
    <el-dialog title="添加分类" @close="editCateDialogClosed" :visible.sync="editCateDialogVisible" width="50%">
      <!-- 编辑修改分类的表单 -->
      <el-form
        ref="editCateFormRef"
        :rules="addCateFormRules"
        :model="editCateForm"
        label-width="90px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        { label: '分类的名称', prop: 'cat_name' },
        {
          label: '是否有效',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列的模板名称
          template: 'order'
        },
        {
          label: '操作',
          //   将当前列定义为模板列
          type: 'template',
          //   表示当前这一列的模板名称
          template: 'operation'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级默认为1级分类
        cat_level: 0
      },
      // 添加分类的表单验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 8, message: '名称长度为2~8个字符', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: [],
      // 控制标记分类对话框的显示与隐藏
      editCateDialogVisible: false,
      // 编辑分类的表单数据对象
      editCateForm: {
        cat_name: ''
      }
    }
  },
  created() {
    this.getData()
  },
  methods: {
    async getData() {
      const { data: res } = await this.$axios.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      // 把数据列表赋值给 catelist
      this.catelist = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getData()
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getData()
    },
    // 点击按钮展示添加分类对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$axios.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      // 如果 this.selectedKeys 数组中的 length 大于0，证明选中了父级分类，否则就没有选中父级分类
      const len = this.selectedKeys.length
      // 父级分类的id
      this.addCateForm.cat_pid = len - 1 >= 0 ? this.selectedKeys[len - 1] : 0
      // 为当前分类的等级赋值
      this.addCateForm.cat_level = len
    },
    // 点击按钮添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$axios.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getData()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 点击展示编辑分类信息对话框
    async showEditCateDialog(cate) {
      console.log(cate)
      const { data: res } = await this.$axios.get('categories/' + cate.cat_id)
      if (res.meta.status !== 200) { return this.$message.error('获取分类信息失败！') }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 监听编辑分类对话框的关闭事件
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) { return }
        const { data: res } = await this.$axios.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name })
        if (res.meta.status !== 200) { return this.$message.error('编辑分类信息失败！') }
        this.getData()
        this.editCateDialogVisible = false
        this.$message.success('编辑成功！')
      })
    },
    // 删除分类
    async deleteCate(cate) {
      console.log(cate)
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') { return }
      const { data: res } = await this.$axios.delete('categories/' + cate.cat_id)
      if (res.meta.status !== 200) { return this.$message.error('删除失败！') }
      this.$message.success('删除成功！')
      this.getData()
    }
  }
}
</script>

<style>
.treeTable {
  margin-top: 15px;
}
</style>
