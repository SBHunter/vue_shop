<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!--表格-->
      <tree-table class="treeTable" :data="catelist" :columns="columns"
        border show-index  index-text="#" :expand-type="false" :selection-type="false">
       <!--是否有效-->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
          <i class="el-icon-error" style="color: red;" v-else></i>
        </template>
        <!--排序-->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" type="success" v-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-if="scope.row.cat_level === 2">三级</el-tag>
        </template>
        <!--操作-->
        <template slot="opt" slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!--分页区域-->
      <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pagenum"
              :page-sizes="[1 , 2, 5, 10]"
              :page-size="queryInfo.pagesize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
      </el-pagination>
    </el-card>
    <!--添加分类的对话框-->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules=" addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
                  v-model="selectedKeys"
                  :options="parentCateList"
                  :props="cascaderProps"
                  clearable
                  @change="parentCateChange"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "cate",
    data() {
      return {
        //查询条件
        queryInfo:{
          type: 3,
          pagenum: 1,
          pagesize: 5
        },
        //商品分类的数据列表，默认为空
        catelist: [],
        //总数据条数
        total: 0,
        //添加分类对话框的显示与隐藏
        addCateDialogVisible: false,
        //为table指定列的定义
        columns:[
          {
            label:'分类名称',
            prop:'cat_name'
          },{
            label:'是否有效',
            //表示将当前列设为模板列
            type: 'template',
            // 表示当前列使用的模板名称
            template:'isok'
          },{
            label:'排序',
            //表示将当前列设为模板列
            type: 'template',
            // 表示当前列使用的模板名称
            template:'order'
          },{
            label:'操作',
            //表示将当前列设为模板列
            type: 'template',
            // 表示当前列使用的模板名称
            template:'opt'
          },
        ],
        addCateForm:{
          //将要添加的分类的名称
          cat_name: '',
          //父级分类的id
          cat_pid: 0,
          //分类的等级默认添加的是一级分类
          cat_level:0
        },
        //添加分类表单的验证规则对象
        addCateFormRules:{
          cat_name:[
            { required: true, message: '请输入分类名称', trigger: 'blur' },
          ]
        },
        parentCateList:[],
        cascaderProps:{
          value:'cat_id',
          label:'cat_name',
          children:'children',
          expandTrigger:'hover',
          checkStrictly:true
        },
        selectedKeys:[],


      }
    },
    created() {
      this.getCateList();
    },
    methods: {
      //获取商品分类数据
      async getCateList() {
        const {data:res} = await this.$http.get('categories', {params:this.queryInfo});
        if(res.meta.status !== 200) return this.$message.error('获取商品分类失败！')
        this.catelist = res.data.result;
        this.total = res.data.total;
      },
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize;
        this.getCateList();
      },
      handleCurrentChange(newNum) {
        this.queryInfo.pagenum = newNum;
        this.getCateList();
      },
      showAddCateDialog() {
        this.getParentCateList();
        this.addCateDialogVisible = true;
      },
      async getParentCateList() {
        this.selectedKeys = [];
        const {data:res} = await this.$http.get('categories', {params:{type:2}});
        if(res.meta.status !== 200) return this.$message.error('获取父级分类数据失败！');
        this.parentCateList = res.data;
        },
      parentCateChange() {
        if(this.selectedKeys.length > 0){
          this.addCateForm.cat_pid = this.selectedKeys[
              this.selectedKeys.length - 1
              ]
          this.addCateForm.cat_level = this.selectedKeys.length
          return
        }else{
          this.addCateForm.cat_pid = 0;
          this.addCateForm.cat_level = 0;
        }
      },
      addCateDialogClosed() {
        this.$refs.addCateFormRef.resetFields();
        this.selectedKeys = [];
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      },
      addCate() {
        this.$refs.addCateFormRef.validate(async valid => {
          if(!valid) return;
          const {data:res} = await this.$http.post('categories', this.addCateForm)
          if(res.meta.status !== 201) return this.$message.error('添加分类失败！')
          this.$message.success('添加分类成功！');
          this.getCateList();
          this.addCateDialogVisible = false;
        })
      }


    },
  }
</script>

<style scoped>
  .el-cascader{
    width: 100%;
  }
</style>