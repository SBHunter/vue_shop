<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <!--警告区域-->
      <el-row>
        <el-col>
          <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon>
          </el-alert>
        </el-col>
      </el-row>
      <!--选择商品分类区域-->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类</span>
          <el-cascader v-model="selectedCateKeys" :options="catelist" :props=" cascaderProps" @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!--Tabs页签区域-->
      <el-tabs v-model="activeName" @tab-click="paneChange">
        <!--添加动态参数的面板-->
        <el-tab-pane label="动态参数" name="many">
          <el-button @click="addDialogVisible = true" type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
          <!--动态参数表格-->
          <el-table :data="manyTableData" border stripe>
            <!--展开行-->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index"  @close="handleClose(index, scope.row)">{{item}}</el-tag>
                <!--输入的文本框-->
                <el-input
                      class="input-new-tag"
                      v-if="scope.row.inputVisible"
                      v-model="scope.row.inputValue"
                      ref="saveTagInput"
                      size="small"
                      @keyup.enter.native="handleInputConfirm(scope.row)"
                      @blur="handleInputConfirm(scope.row)"
              >
              </el-input>
                <!--添加的按钮-->
               <el-button v-else size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!--索引列-->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!--添加静态属性的面板-->
        <el-tab-pane label="静态属性" name="only">
          <el-button @click="addDialogVisible = true" type="primary" size="mini" :disabled="isBtnDisabled">添加属性</el-button>
          <!--静态属性表格-->
          <el-table :data="onlyTableData" border stripe>
            <!--展开行-->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index"@close=" handleClose(index, scope.row)">{{item}}</el-tag>
                <!--输入的文本框-->
                <el-input
                        class="input-new-tag"
                        v-if="scope.row.inputVisible"
                        v-model="scope.row.inputValue"
                        ref="saveTagInput"
                        size="small"
                        @keyup.enter.native="handleInputConfirm(scope.row)"
                        @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!--添加的按钮-->
                <el-button v-else size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!--索引列-->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini"  @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!--添加参数的对话框-->
    <el-dialog @close="addDialogClosed" :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%">
      <!--添加参数的对话框-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!--修改参数的对话框-->
    <el-dialog @close="editDialogClosed" :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%">
      <!--添加参数的对话框-->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    name: "params",
    created() {
      this.getCateList();
    },
    data() {
      return {
        //商品分类列表
        catelist:[],
        selectedCateKeys:[],
        cascaderProps:{
          expandTrigger: 'hover',
          children:'children',
          value:'cat_id',
          label:'cat_name'
        },
        activeName:'many',
        manyTableData:[],
        onlyTableData:[],
        addDialogVisible: false,
        editDialogVisible: false,
        //添加参数的表单数据对象
        addForm:{
          attr_name:''
        },
        addFormRules:{
          attr_name:[
              { required: true, message: '请输入活动名称', trigger: 'blur' },
          ],
        },
        editForm:{
          attr_name:''
        },
        editFormRules:{
          attr_name:[
            { required: true, message: '请输入活动名称', trigger: 'blur' },
          ],
        },
        inputVisible:false,
        inputValue:'',


      }
    },
    methods:{
      //获取所有商品的分类
      async getCateList() {
        const {data:res} = await this.$http.get('categories')
        if(res.meta.status !== 200) return this.$message.error('获取商品分类失败！')
        this.catelist = res.data;
      },
      async getParamsData() {
        //  根据所选分类的id，和当前所处的面板，获取对应的参数
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`, {params:{sel:this.activeName}})
        if(res.meta.status != 200) return this.$message.error('获取参数列表失败');
        res.data.forEach(item => {
          item.attr_vals  = item.attr_vals ? item.attr_vals.split(' '): [];
          item.inputVisible = false;
          item.inputValue = '';
        })
        if(this.activeName === 'many'){
          this.manyTableData = res.data;
        }else{
          this.onlyTableData = res.data;
        }
      },
      async handleChange() {
        if(this.selectedCateKeys.length !== 3){
          this.selectedCateKeys = [];
          this.manyTableData = [];
          this.onlyTableData = [];
          return;
        }
        this.activeName = 'many'
        this.getParamsData();
      },
      paneChange() {
        this.getParamsData();
      },
      //监听添加对话框的关闭事件
      addDialogClosed() {
        this.$refs.addFormRef.resetFields();
      },
      //点击按钮，添加参数
      addParams() {
        this.$refs.addFormRef.validate(async valid => {
          if(!valid) return;
          const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,
              {attr_name:this.addForm.attr_name, attr_sel:this.activeName});
          if(res.meta.status !== 201) return this.$message.error('添加参数失败！');
          this.$message.success('添加参数成功！');
          this.addDialogVisible = false;
          this.getParamsData();
        });
      },
      //点击按钮展示修改对话框
      async showEditDialog(attr_id) {
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, {params:{attr_sel:this.activeName}})
        if(res.meta.status !== 200) return this.$message.error('获取参数信息失败！');
        this.editForm = res.data;
        this.editDialogVisible = true;
      },
      //重置修改表单
      editDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      editParams() {
        this.$refs.editFormRef.validate(async valid => {
          if(!valid) return
          const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {attr_name:this.editForm.attr_name,attr_sel:this.activeName})
          if(res.meta.status !== 200) return this.$message.error('修改参数失败！');
          this.$message.success('修改参数成功！');
          this.getParamsData();
          this.editDialogVisible  = false;
        })
      },
      async removeParams(attr_id) {
        const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        //用户取消了删除操作
        if(confirmResult !== 'confirm') return this.$message.info('取消了删除操作');
        //删除业务逻辑
        const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`);
        if(res.meta.status !== 200){
          return this.$message.error('删除参数失败！')
        }
        this.$message.success('删除参数成功！')
        this.getParamsData();
      },
      async handleInputConfirm(row){
        if(row.inputValue.trim().length === 0){
          row.inputValue = '';
          return
        }
      //  如果没有return，则证明输入的内容，需要做后续处理
        row.attr_vals.push(row.inputValue.trim())
        row.inputValue = '';
        this.saveAttrVals(row);

        row.inputVisible = false;
      },
      showInput(row) {
        row.inputVisible = true;
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      },
      handleClose(i,row) {
        row.attr_vals.splice(i, 1)
        this.saveAttrVals(row)
      },
      async saveAttrVals(row) {
        const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
            {attr_name:row.attr_name, attr_sel:row.attr_sel, attr_vals:row.attr_vals.join(' ')});
        if(res.meta.status !== 200) return this.$message.error('修改参数项失败！');
        this.$message.success('添加参数项成功！');
      },


    },
    computed: {
      isBtnDisabled() {
        return this.selectedCateKeys.length !== 3;
      },
      //当前选中的三级分类id
      cateId() {
        if(this.selectedCateKeys.length == 3) return this.selectedCateKeys[2];
        return null;
      },
      //动态计算标题的文本
      titleText() {
        if(this.activeName === 'many') return '动态参数';
        return '静态属性'
      },

    },

  }
</script>

<style scoped>
.cat_opt{
  margin: 15px;
}
  .el-tag{
    margin: 5px;
  }
  .input-new-tag{
    width: 150px;
  }
</style>