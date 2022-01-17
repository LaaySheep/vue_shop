<template>
    <div>
        <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 商品分类内容主体部分 -->
    <el-card class="box-card">
      <!-- layout布局，:gutter表示间距，:span宽度 -->
      <el-row :gutter="20">
        <el-col :span="7">
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="showCatedialog"
            >添加商品</el-button
          >
        </el-col>
      </el-row>
      <!--商品分裂表格区域 -->
      <tree-table :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
      <!-- 是否有效-->
        <template slot-scope="scope" slot="isok">
          <i class="el-icon-success" v-if="scope.row.cat_delete !== false" style="color: green"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
         <!-- 排序 -->
         <template slot-scope="scope" slot="order">
         <el-tag v-if="scope.row.cat_level === 0" size="mini">标签一</el-tag>
         <el-tag type="success" v-else-if="scope.row.cat_level === 1" size="mini">标签二</el-tag>
         <el-tag type="warning" v-else-if="scope.row.cat_level === 2"  size="mini">标签三</el-tag>
        </template>
      </tree-table>
      <!--分页区域-->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
     title="添加分类"
     :visible.sync="addCatedialogVisible"
     width="50%">
     <!--添加分类的表单-->
     <el-form :model="addCateForm" :rules="addCateFormrules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
     <el-form-item label="分类名称:" prop="cat_name">
       <el-input v-model="addCateForm.cat_name"></el-input>
     </el-form-item>
     <el-form-item label="父级分类:">
     <!--级联选择器 option表示数据源-->
    <el-cascader
      expand-trigger="hover"
      v-model="selectKeys"
      :options="parentCateList"
      :props="cascaderProps"
      @change="parentCateChange"></el-cascader>
     </el-form-item>
     </el-form>
     <span slot="footer" class="dialog-footer">
       <el-button @click="addCatedialogVisible = false">取 消</el-button>
       <el-button type="primary" @click="addCatedialogVisible = false">确 定</el-button>
     </span>
    </el-dialog>
    </div>
</template>
<script>
export default {
  data() {
    return {
      // 指定查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类列表数组，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      addCatedialogVisible: false,
      // 为table指定列的定义
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '是否有效', type: 'template', template: 'isok' },
        // 定义当前为模板列，isok表示模板名称
        { label: '排序', type: 'template', template: 'order' }
      ],
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 父级id
        cat_pid: 0,
        // 默认添加一级分类
        cat_level: 0
      },
      addCateFormrules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类列表
      parentCateList: [],
      // 指定级联配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类id数组
      selectKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      console.log(res.data)
      // 给catelist赋值
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 点击按钮展示添加分类对话框
    showCatedialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      this.addCatedialogVisible = true
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败!')
      }
      console.log(res)
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChange() {
      console.log(this.selectKeys)
      if (this.selectKeys > 0) {
        // 父级分类的id
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        // 当前分类等级的赋值
        this.addCateForm.cat_level = this.selectKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = this.selectKeys.length
      }
    }
  }
}

</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
