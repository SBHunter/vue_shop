<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-button type="primary">添加角色</el-button>
      <el-table :data="rolesList" border>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id"  :class="['vcenter','bdbottom', i1 == 0 ? 'bdtop' : 'bdbottom']">
              <!--渲染一级权限-->
              <el-col :span="5">
                <el-tag closable  @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--渲染二级和三级权限-->
              <el-col :span="19">
                <!--通过for循环渲染二级权限-->
                <el-row v-for="(item2, i2) in item1.children" :class="[i2 == 0 ? '': 'bdtop','vcenter']" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!--渲染三级权限-->
                  <el-col :span="18">
                    <el-tag @close="removeRightById(scope.row, item3.id)" closable v-for="(item3, i3) in item2.children" :key="item3.id" type="warning">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column prop="level" label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--分配权限的对话框-->
    <el-dialog
            title="分配权限"
            :visible.sync="setRightsDialogVisible"
            width="50%"
    >
      <!--树形控件-->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox
               ref="treeRef" :default-checked-keys="defKeys" default-expand-all node-key="id"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightsDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="alterRights">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "roles",
    created(){
      this.getRolesList()
    },
    data() {
      return {
        rolesList:[],
        setRightsDialogVisible: false,
        rightsList:[],
        treeProps:{
          label:'authName',
          children:'children'
        },
        //默认选中的结点id值数组
        defKeys:[],
        roleId: ''
      }
    },
    methods: {
      async getRolesList() {
        const {data:res} = await this.$http.get('roles');
        if(res.meta.status !== 200) return this.$message.error('获取角色列表失败');
        this.rolesList = res.data;
      },
      async removeRightById(role, rightId) {
        const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') return this.$message.info('取消了删除')
        const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message.success('删除成功')
        role.children = res.data;
      },
      //展示分配权限的对话框
      async showSetRightDialog(node) {
        //获取所有权限的数据
        const {data:res} = await this.$http.get('rights/tree');
        if(res.meta.status !== 200) return this.$message.error('获取权限失败');
        this.rightsList = res.data;
        this.defKeys = [];
        this.getLeafKeys(node, this.defKeys);
        this.roleId = node.id;
        this.setRightsDialogVisible = true;

      },
      //通过递归的的形式，获取角色下所有的三级权限的id，并保存到defKeys数组中
      getLeafKeys(node,arr) {
        if(!node.children){
          return arr.push(node.id);
        }
        node.children.forEach(item => {
          this.getLeafKeys(item,arr);
        })
      },
      async alterRights() {
        const rightIds = [
            ...this.$refs.treeRef.getCheckedKeys(),
            ...this.$refs.treeRef.getHalfCheckedKeys()
        ];
        const rightIdsStr = rightIds.join(',');
        const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids:rightIdsStr});
        if(res.meta.status !== 200) return this.$message.error('分配权限失败');
        this.$message.success('分配权限成功');
        console.log(res);
        this.setRightsDialogVisible = false;
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
  .vcenter{
    display: flex;
    align-items: center;
  }

</style>