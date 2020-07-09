<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!--Table表格区域-->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格（元）" width="120px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="120px"></el-table-column>
        <el-table-column label="创建时间" width="170px">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"  width="160px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页区-->
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
  </div>
</template>

<script>
  export default {
    name: "list",
    created() {
      this.getGoodsList();
    },
    data() {
      return{
        //查询参数对象
        queryInfo:{
          query: '',
          pagenum: 1,
          pagesize:10
        },
        //商品列表
        goodslist:[],
        //总数据条数
        total:0,

      }
    },
    methods: {
      async getGoodsList(){
        const {data:res} = await this.$http.get('goods', {params:this.queryInfo});
        if(res.meta.status !== 200) return this.$message.error('获取商品列表失败！');
        this.goodslist = res.data.goods;
        this.total = res.data.total;
      },
      handleSizeChange(size) {
        this.queryInfo.pagesize = size;
        this.getGoodsList();
      },
      handleCurrentChange(num) {
        this.queryInfo.pagenum = num;
        this.getGoodsList();
      },
      async removeById(row){
        const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') return this.$message.info('已取消删除！');
        const {data:res} = await this.$http.delete(`goods/${row.goods_id}`);
        console.log(res.data);
        if(res.meta.status !== 200) return this.$message.error('删除商品失败！');
        this.getGoodsList();

      },
      goAddpage() {
        this.$router.push('/goods/add')
      },

    },
  }
</script>

<style scoped>

</style>