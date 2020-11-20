<template>
<div>
  <!-- 面包屑导航区域 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home'}">首页</el-breadcrumb-item>
    <el-breadcrumb-item>权限管理</el-breadcrumb-item>
    <el-breadcrumb-item>角色列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图区 -->
  <el-card>
    <!-- 添加角色按钮区域 -->
    <el-row :gutter="20">
<!--      &lt;!&ndash; 搜索角色区域 &ndash;&gt;-->
<!--      <el-col :span="8">-->
<!--        <el-input placeholder="请输入用户名" v-model="" clearable @clear="getRolesList">-->
<!--          <el-button slot="append" icon="el-icon-search" @click="getRolesList"></el-button>-->
<!--        </el-input>-->
<!--      </el-col>-->
      <!-- 添加角色按钮 -->
      <el-col :span="4">
        <el-button type="primary" @click="addRoleDialog">添加角色</el-button>
      </el-col>
    </el-row>
    <!-- 角色列表区域 -->
    <el-table :data="rolelist" border stripe>
      <!-- 展开列 -->
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
            <!-- 渲染一级权限-->
            <el-col :span="5">
              <el-tag closable @close="removeRigById(scope.row, item1.id)"> {{item1.authName}} </el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- 渲染二和三级权限-->
            <el-col :span="19">
              <!-- 通过嵌套for循环渲染二级权限-->
              <el-row :class="[i2 ===0 ? '': 'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                <el-col :span="6">
                  <el-tag type="success" closable @close="removeRigById(scope.row, item2.id)"> {{item2.authName}} </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 三级权限 -->
                <el-col :span="18">
                  <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRigById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" width="300px">
        <template slot-scope="scope">
          <!-- 修改按钮 -->
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
          <!-- 删除按钮 -->
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
          <!-- 分配角色按钮 -->
          <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
  <!-- 分配权限对话框 -->
  <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
    <!-- 树形控件 -->
    <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
    <span slot="footer" class="dialog-footer">
      <el-button @click="setRightDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="allotRights">确 定</el-button>
    </span>
  </el-dialog>
  <!-- 展示编辑角色对话框 -->
  <el-dialog
    title="编辑角色"
    :visible.sync="showEditDialogVisible"
    width="50%"
    @close="editClosed"
  >
    <div>
      <el-form :model="roleEditForm" ref="roleEditFormRef" label-width="90px" :rules="roleEditFormRules">
        <el-form-item label="角色名称" prop="rolename">
          <el-input v-model="roleEditForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roledesc">
          <el-input v-model="roleEditForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
    </div>
    <span slot="footer" class="dialog-footer">
            <el-button @click="showEditDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="showEditInfo">确 定</el-button>
          </span>
  </el-dialog>
  <!--添加角色的对话框 -->
  <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleClosed">
    <el-form :model="addRole" :rules="addRoleRules" ref="addRoleRef" label-width="70px">
      <el-form-item label="角色名称" prop="rolename">
        <el-input v-model="addRole.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roledesc">
        <el-input v-model="addRole.roleDesc"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="addRoleDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addRoles">确 定</el-button>
    </span>
  </el-dialog>
</div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表数据
      rolelist: [],
      //控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      //控制编辑对话框的显示与隐藏
      showEditDialogVisible: false,
      //所有权限数据
      rightslist: [],
      //树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //默认选中的节点ID值数组
      defKeys: [],
      //即将分配权限的id
      roleId: '',
      //控制添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      //添加角色表单数据
      addRole: {
        roleName: '',
        roleDesc: ''
      },
      //添加角色验证规则
      addRoleRules: {
        rolename: [
          // {
          //   required: true,
          //   message: '请输入角色名',
          //   trigger: 'blur'
          // },
          {
            min: 3,
            max: 10,
            message: '长度在2到10个字符',
            trigger: 'blur'
          }
        ],
        roledesc: [
          { min: 0, max: 50, message: '角色描述的长度在50个字符以内', tigger: 'blur' }
        ]
      },
      // 编辑角色的信息
      roleEditForm: {},
      // 编辑角色rules
      roleEditFormRules: {
        rolename: [
          // { required: true, message: '请输入角色名称', tigger: 'blur' },
          { min: 2, max: 10, message: '角色名称的长度在2~10个字符之间', tigger: 'blur' }
        ],
        roledesc: [
          { min: 0, max: 50, message: '角色描述的长度在50个字符以内', tigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const {
        data: res
      } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
      //console.log(this.rolelist)
    },

    //监听添加对话框关闭的事件
    addRoleClosed() {
      this.$refs.addRoleRef.resetFields()
    },
    //点击添加角色的确定按钮添加新角色
    async addRoles(){

      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return
        //可以发起添加用户的网络请求
        const {
          data: res
        } = await this.$http.post('roles', this.addRole)
        console.log(res)
        if (res.meta.status !== 201) {
          this.$message.error('添加角色失败！')
          console.log(res.meta.status)
        }
        this.$message.success('添加角色成功！')
        this.addRoleDialogVisible = false
        this.getRolesList()
      })
    },



    // 展示编辑角色对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败！')
      }
      this.roleEditForm = res.data
      console.log(this.roleEditForm)
      this.showEditDialogVisible = true
    },
    // 监听编辑对话框重置
    editClosed () {
      this.$refs.roleEditFormRef.resetFields()
    },
    // 编辑角色信息提交
    showEditInfo () {
      this.$refs.roleEditFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.roleEditForm.roleId, { roleName: this.roleEditForm.roleName, roleDesc: this.roleEditForm.roleDesc })
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色失败！')
        }
        this.showEditDialogVisible = false
        this.getRolesList()
        return this.$message.success('更新角色成功！')
      })
    },
    //根据id删除对应的权限
    async removeRigById(role, rightId) {
      //弹框提示用户是否删除
      const confirmResult =
        await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除权限')
      }

      const {
        data: res
      } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }

      // this.getRolesList()
      role.children = res.data

    },
    //展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      //获取所有权限的数据
      const {
        data: res
      } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      //获取到的权限数据保持到data中
      this.rightslist = res.data
      //console.log(this.rightslist)
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    //通过递归的形式获取角色下三级id
    //node节点
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    //监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    //点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const {
        data: res
      } = await this.$http.post(`roles/${this.roleId}/rights`, {
        rids: idStr
      })
      if (res.meta.status !== 200) {
        return this.$message.error('分配用户权限失败')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    addRoleDialog() {
      this.addRoleDialogVisible = true
    },

    // 删除角色
    async removeRoleById(id){
      // 弹框
      const confirmResult = await this.$confirm('此操作将永久删除该角色， 是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })
      // 如果用户确认删除，则返回的字符串为confirm
      // 如果用户取消删除，则返回的字符串为cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除用户成功！')
      this.getRolesList()
    }
  },
}
</script>

<style lang="less" scoped>
.el-tag {
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
