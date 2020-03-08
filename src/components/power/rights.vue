<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template v-slot="scope">
            <el-tag :type="tagState[scope.row.level].type">{{tagState[scope.row.level].name}}</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rightsList: [],
      tagState: [
        { type: '', name: '一级' },
        { type: 'success', name: '二级' },
        { type: 'warning', name: '三级' }
      ]
    }
  },
  created() {
    this.getData()
  },
  methods: {
    async getData() {
      const { data: res } = await this.$axios.get('rights/list')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      this.rightsList = res.data
      console.log(this.rightsList)
    }
  }
}
</script>

<style>
</style>
