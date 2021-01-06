<template>
    <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/Welcome' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            </el-col>
        </el-row>
        <!-- 表格 -->
        <tree-table class="treeTable" :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index index-text="#" border
        :show-row-hover="false">
        <template slot="isok" slot-scope="scope">
            <i class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"></i>
            <i class="el-icon-error" v-else
            style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
            <el-tag size="mini" type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
            <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <!-- 删除按钮 -->
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteDialog(scope.row.id)">删除</el-button>
        </template>
        </tree-table>
        <!-- 分页区域 -->
        <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
        </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
    title="添加分类"
    :visible.sync="setDialogVisible"
    width="50%"
    @close="addCateDialogClosed">
    <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
      <el-form-item label="分类名称：" prop="cat_name">
        <el-input v-model="addCateForm.cat_name"></el-input>
      </el-form-item>
      <el-form-item label="父级类别：">
        <el-cascader
        v-model="selectedKeys" :props="cascaderProps"
        :options="parentlist"
        @change="parentCateChange" clearable change-on-select>
        </el-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="setDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate" >确 定</el-button>
    </span>
    </el-dialog>

    </div>
</template>
<script>
export default {
  data() {
    return {
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //   商品分类的数据列表，默认为空
      catelist: [],
      //    总数据条数
      total: 0,
      //   为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //   表示当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'isok'
        },
        {
          label: '排序',
          //   表示当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          //   表示当前列定义为模板列
          type: 'template',
          //   表示当前这一列使用模板名称
          template: 'opt'
        }
      ],
      //   控制添加分类对话框的显示与隐藏
      setDialogVisible: false,
      //   添加分类的表单数据对象
      addCateForm: {
        //   将要添加的分类的名称
        cat_name: '',
        cat_pid: 0,
        // 分类的等级默认添加的为一级
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //   父级数据列表
      parentlist: [],
      //   指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //   选中选择器的父级分类的id
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      // 把数据列表赋值给catelist
      this.catelist = res.data.result
      console.log(res.data)
      this.total = res.data.total
    },
    //  监听pagesize改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 展开添加对话框
    showAddCateDialog() {
      //  获取父级分类的数据列表
      this.getParentList()
      //   再展示出对话框
      this.setDialogVisible = true
    },
    // 获取父级类别数据列表
    async getParentList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据列表失败')
      }
      console.log(res.data)
      this.parentlist = res.data
    },
    parentCateChange() {
      console.log(this.selectedKeys)
      //   如果selectedKeys的length大于0，证明选中了父级分类
      //   反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        //   父级分类的ID (最后一项的索引)
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        //   为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        //   为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // /点击按钮，添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async vaild => {
        if (!vaild) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.setDialogVisible = false
      })
    },
    // 关闭对话框事件,重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 删除事件
    async deleteDialog(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户，是否继续？', '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmResult)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消了删除，则返回值为 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getCateList()
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}
</style>
