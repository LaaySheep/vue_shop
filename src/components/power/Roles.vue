<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>权限管理</el-breadcrumb-item>
          <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-row>
                <el-col><el-button type="primary"  @click="addRolesDialogVisible = true">添加角色</el-button></el-col>
            </el-row>
            <el-table :data="roleList" border style="width: 100%" stripe>
            <!--展开列-->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-row :class="['bdbottom' , i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                  <!-- 渲染一级权限 -->
                  <el-col :span="5">
                    <el-tag>{{item1.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染二级和三级权限 -->
                  <el-col :span="19">
                   <!-- 通过for循环 嵌套 渲染二级权限 -->
                   <el-row :class="[ i2 === 0 ? '': 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id" >
                     <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                     </el-col>
                     <!-- 渲染三级权限 -->
                     <el-col :span="13">
                      <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                        {{item3.authName}}
                      </el-tag>
                     </el-col>
                   </el-row>
                  </el-col>
                </el-row>
              </template>
            </el-table-column>
            <!-- 索引列 -->
                <el-table-column type="index" label="#" width="80"></el-table-column>
                <el-table-column prop="roleName" label="角色名称"></el-table-column>
                <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
                <el-table-column prop="level" label="操作">
                    <template slot-scope="scope">
                        <el-button type="primary" class="el-icon-edit" size="mini">编辑</el-button>
                        <el-button type="danger" class="el-icon-delete" size="mini" @click="removeRolesById(scope.row.id)">删除</el-button>
                        <el-button type="warning" class="el-icon-setting"  size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
        </el-table-column>
    </el-table>
  </el-card>
  <!-- 分配权限对话框 -->
  <el-dialog
    title="分配权限"
    :visible.sync="setrightDialogVisible"
    width="30%"
    :data="rightlist"
    @close="setrightDialogClosed"
    >
      <el-tree
       :data="rightlist"
       show-checkbox
       node-key="id"
       :props="treeProps"
       default-expand-all
       :default-checked-keys="defKeys"
       ref="treeRef"
       >
      </el-tree>
    <span slot="footer" class="dialog-footer">
    <el-button @click="setrightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
    </span>
  </el-dialog>

  <!-- 添加角色对话框 -->
  <el-dialog title="添加角色" :visible.sync="addRolesDialogVisible" width="50%">
      <!-- 内容主体区域 -->
      <template>
          <el-form
        label-width="80px"
        :model="addRolesForm"
        :rules="addRolesRules"
        ref="addRolesFormRef"
        >
        <el-form-item label="角色名称" prop="authName" >
          <el-input v-model="addRolesForm.authName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRolesForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles"
          >确 定</el-button
        >
      </span>
      </template>
    </el-dialog>
</div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表
      roleList: [],
      setrightDialogVisible: false,
      addRolesDialogVisible: false,
      // 所有权限的数据
      rightlist: [],
      addRolesForm: {
        authName: '',
        roleDesc: ''
      },
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认保存权限id的数组
      defKeys: [],
      addRolesRules: {
        authName: [{ required: true, message: '请输入角色名称', tigger: 'blur' },
          { min: 2, max: 6, message: '字符串长度在2~6之间', tigger: 'blur' }
        ]
      },
      // 当前即将分配权限的id
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      this.roleList = res.data
    },
    // 根据id删除对应权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否要删除权限
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      // 重新获取数据
      role.children = res.data
      // this.getRoleList()
      return this.$message.success('成功删除该权限')
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')

      if (res.meta.status !== 200) {
        return this.$$message.error('获取权限失败')
      }
      // 将获取到的数据保存到data中
      this.rightlist = res.data

      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)

      this.setrightDialogVisible = true
    },
    // 获取该角色所拥有的权限
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    // 监听权限对话框的关闭事件
    setrightDialogClosed() {
      this.defKeys = []
    },
    // 添加角色
    addRoles() {
      this.$refs.addRolesFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        // 发送请求，根据status的状态来判断是否添加成功
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 隐藏对话框
        this.addRolesDialogVisible = false
        // 重新获取用户列表
        this.getRoleList()
      })
    },
    // 点击角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配角色权限失败')
      }
      this.$message.success('分配角色权限成功')
      this.getRoleList()
      this.setrightDialogVisible = false
    },
    async removeRolesById(id) {
      // 弹窗询问用户是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }
      ).catch(err => err)
      // 如果用户确认删除，则返回字符串confirm。如果取消删除，返回字符串cancel
      console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('成功删除用户')
      this.getUserList()
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag{
margin: 10px;
}
.bdtop{
border-top: 1px solid #eee;
}
.bdbottom{
border-bottom: 1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center
}
</style>
