<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图-->
    <el-card class="box-card">
        <el-table :data="rightsList" border style="width: 100%" stripe>
        <el-table-column type="index" label="#" width="80"></el-table-column>
        <el-table-column prop="authName" label="权限名称"></el-table-column>
        <el-table-column prop="path" label="路径"></el-table-column>
        <el-table-column prop="level" label="权限等级">
            <template slot-scope="scope">
                <el-tag v-if="scope.row.level === '0'" type="success">一级</el-tag>
                <el-tag v-else-if="scope.row.level === '1'" type="warning">二级</el-tag>
                <el-tag v-if="scope.row.level === '2'" type="error">三级</el-tag>
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
      rightsList: []
    }
  },
  created() {
    this.getRightsList()
  },
  methods: {
    async getRightsList() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表信息失败')
      }
      this.rightsList = res.data
      console.log(this.rightsList)
    }
  }
}
</script>
<style lang="less" scoped>

</style>
