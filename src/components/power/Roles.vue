<template>
    <div>
     <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 添加角色按钮区域 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表区域 -->
        <el-table :data="rolesList" border stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
                    v-for=" (item1,i1) in scope.row.children" :key="item1.id" >
                        <!-- 渲染一级权限 -->
                         <el-col :span="5">
                            <el-tag closable
                                     @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                         </el-col>
                         <!-- 渲染二级权限 -->
                         <el-col :span="19">
                             <el-row :class="[ i2 === 0 ? '' : 'bdtop', 'vcenter']"
                             v-for="(item2,i2) in item1.children" :key="item2.id" closable
                                     @close="removeRightById(scope.row, item2.id)">
                                 <el-col :span="6">
                                     <el-tag type="success" closable
                                     @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                     <i class="el-icon-caret-right"></i>
                                 </el-col>
                                 <el-col :span="18">
                                     <el-tag type="warning" :class="[ i3 === 0 ? '' : 'bdtop']" v-for="(item3,i3) in
                                     item2.children" :key="item3.id" closable
                                     @close="removeRightById(scope.row, item3.id)">
                                         {{item3.authName}}
                                     </el-tag>
                                 </el-col>
                             </el-row>
                         </el-col>
                    </el-row>
                    <pre>
                        <!-- {{scope.row}} -->
                    </pre>
                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" width="300px" >
            <template slot-scope = "scope" >
                <!-- 修改按钮 -->
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEdiDialog(scope.row.id)">修改</el-button>
                <!-- 删除按钮 -->
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteDialog(scope.row.id)">删除</el-button>
                <!-- 分配角色 -->
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
        </el-table-column>
        </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%"
    @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!--添加角色对话框 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="addDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addRoleInfo">确 定</el-button>
    </span>
    </el-dialog>
    <!--修改角色对话框  -->
    <el-dialog
    title="修改角色"
    :visible.sync="ediDialogVisible" width="50%">
      <el-form :model="ediForm" :rule="ediFormRules" ref="ediFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="ediForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="ediForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="ediDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="ediRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolesList: [],
      //  控制对话框的显示与隐藏
      setRightDialogVisible: false,
      //   所有权限的数据
      rightslist: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 获取三级权限的id
      defKeys: [],
      roleId: '',
      // 修改角色对话框的展示与掩藏
      ediDialogVisible: false,
      // 添加角色对话框的展示与掩藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加用户的验证规则对象
      addFormRules: {
        roleName: [
          {
            required: true,
            message: '请输入角色名',
            trigger: 'blur'
          },
          {
            min: 2,
            max: 10,
            message: '角色名的长度在3~10字符之间',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          },
          {
            min: 2,
            max: 10,
            message: '角色描述的长度在3~10字符之间',
            trigger: 'blur'
          }
        ]
      },
      // 查询到用户信息对象
      ediForm: {},
      // 修改用户的验证规则对象
      ediFormRules: {
        roleName: [
          {
            required: true,
            message: '请输入角色名',
            trigger: 'blur'
          },
          {
            min: 2,
            max: 10,
            message: '角色名的长度在3~10字符之间',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            trigger: 'blur'
          },
          {
            min: 2,
            max: 10,
            message: '角色描述的长度在3~10字符之间',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    //  获取所有权限
    this.getRolesList()
  },
  methods: {
    //   获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rolesList = res.data
      // console.log(this.rolesList)
    //   this.$message.success('更新用户状态成功')
    },
    //  根据id删除对应的权限
    async removeRightById(role, rightId) {
    //  弹框提示用户是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除！')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      //   this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }

      // 把获取到的权限数据保存到 data 中
      this.rightslist = res.data
      // console.log(this.rightslist)

      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归将分配权限的第三节点id传到defKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    // 分配权限对话框清空
    setRightDialogClosed() {
      this.defKeys = []
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr })
      if (res.meta.status !== 200) {
        // console.log(this.res.meta.status)
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 展示修改用户对话框
    async showEdiDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户信息失败！')
      }
      this.ediForm = res.data
      this.ediDialogVisible = true
    },
    // 添加角色信息并提交
    addRoleInfo() {
      this.$refs.addFormRef.validate(async vaild => {
        if (!vaild) return false
        // 发起添加角色信息的数据请求
        const { data: res } = await this.$http.post(
          'roles',
          {
            roleName: this.addForm.roleName,
            roleDesc: this.addForm.roleDesc
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色信息失败！')
        }
        // 关闭对话框
        this.addDialogVisible = false
        // 刷新列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('添加角色信息成功')
      })
    },
    // 修改角色信息并提交
    ediRoleInfo() {
      this.$refs.ediFormRef.validate(async valid => {
        if (!valid) return false
        // 发起修改请求
        const { data: res } = await this.$http.put(
          'roles/' + this.ediForm.roleId,
          {
            roleName: this.ediForm.roleName,
            roleDesc: this.ediForm.roleDesc
          })
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色信息失败！')
        }
        // 关闭对话框
        this.ediDialogVisible = false
        // 刷新列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('修改角色信息成功！')
      })
    },
    // 删除用户的事件
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('用户删除失败！')
      }
      this.$message.success('用户删除成功！')
      this.getRolesList()
    }
  }
}

</script>
<style lang="less">
 .el-tag{
     margin: 7px;
 }
 .bdtop {
     border-top: 1px solid #eee;
 }
 .bdbottom {
     border-bottom: 1px solid #eee;
 }
 .vcenter {
    display: flex;
    align-items: center;
 }
</style>
