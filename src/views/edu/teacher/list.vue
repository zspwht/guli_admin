<template>
  <div class="app-container">
    <!--查询表单 -->
    <el-form :inline="true" class="demo-form-inline">
        <el-form-item>
            <el-input v-model="searchobj.name" placeholder="讲师名"/>
        </el-form-item>
        <el-form-item>
            <el-select v-model="searchobj.level" clearable placeholder="讲师头衔">
                <el-option :value="1" label="高级讲师"></el-option>
                <el-option :value="2" label="首席讲师"></el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="添加时间">
            <el-form-item>
                <el-date-picker v-model="searchobj.beginTime" 
                type="datetime" placeholder="请选择开始时间"
                value-format="yyyy-MM-dd HH:mm:ss"/>
            </el-form-item>
            <el-form-item>
                <el-date-picker v-model="searchobj.endTime" 
                type="datetime" placeholder="请选择结束时间"
                value-format="yyyy-MM-dd HH:mm:ss" />
            </el-form-item>
        </el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="fetchData()">查询</el-button>
        <el-button type="default"  @click="restDate()">清空</el-button>
    </el-form>
    <!--表格 -->
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="数据加载中"
      border
      fit
      highlight-current-row
    >
      <el-table-column label="序号" width="70" align="center">
        <template slot-scope="scope">{{
          (page - 1) * limit + scope.$index + 1
        }}</template>
      </el-table-column>
      <el-table-column prop="name" label="名称" width="80" />
      <el-table-column label="头衔" width="80">
        <template slot-scope="scope">
          {{ scope.row.level == 1 ? "高级讲师" : "首席讲师" }}
        </template>
      </el-table-column>
      <el-table-column prop="intro" label="资历" />
      <el-table-column prop="gmtCreate" label="添加时间" width="160" />
      <el-table-column prop="sort" label="排序" width="60" />
      <el-table-column label="操作" width="200" align="center">
        <template slot-scope="scope">
          <router-link :to="'/edu/teacher/edit/' + scope.row.id">
            <el-button type="primary" size="mini" icon="el-icon-edit"
              >修改</el-button
            >
          </router-link>
          <el-button
            type="danger"
            size="mini"
            icon="el-icon-delete"
            @click="removeDataById(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!--分页 -->
    <el-pagination :current-page="page" :page-size="limit" :total="total"
        style="padding:30px 0;text-align:center" layout="total,prev,pager,next,jumper" 
        @current-change="fetchData"/>
  </div>
</template>
<script>
export default {
  data() {
    return {
      listLoading: true,
      list: null,
      total: 0,
      page: 1,
      limit: 3,
      searchobj: {
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData(page=1,limit=3) {
      console.log("加载列表");
      //debugger;
    //   this.page = page;
    //   this.limit = this.limit;
      this.listLoading = true;
      this.$axios({
        url:`http://localhost:8885/admin/edu/teacher/${page}/${limit}`,
        method: "get",
        params:this.searchobj
      })
        .then((res) => {
          //debugger;
          console.log(res.data.data.rows);
          this.list = res.data.data.rows;
          this.total = res.data.data.total;
          this.listLoading = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    removeDataById(id) {
      this.$confirm("是否删除？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(
          this.$axios({
            url: `http://localhost:8885/admin/edu/teacher/${id}`,
            method: "delete",
          }).then((res) => {
            const result = res.data;
            if (result.code == 20000) {
              console.log("删除成功");
            }
          })
        )
        .then(() => {
          this.fetchData();
          this.$message({
            type: "success",
            message: "删除成功！",
          });
        })
        .catch((response) => {
          if (response == "cancel") {
            this.$message({
              type: "info",
              message: "已取消删除",
            });
          } else {
            this.$message({
              type: "error",
              message: "删除失败",
            });
          }
        });
    },
    restDate(){
        this.searchobj = {};
        this.fetchData();
    }
  },
};
</script>