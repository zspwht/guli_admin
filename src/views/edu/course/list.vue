<template>
  <div class="app-container">
    <!--查询表单-->
    <el-form :inline="true" class="demo-from-inline">
      <el-form-item label="课程类别">
        <el-select v-model="searchObj.subjectParentId" placeholder="请选择" @change="subjectLeaveOneChaged">
          <el-option v-for="subject in subjectNestedList" :key="subject.id" :label="subject.title" :value="subject.id"/>
        </el-select>
        <el-select v-model="searchObj.subjectId" placeholder="请选择">
          <el-option v-for="subject in subSubjectList" :key="subject.id" :label="subject.title" :value="subject.id"/>
        </el-select>
      </el-form-item>
      <el-form-item label="课程标题">
        <el-input v-model="searchObj.title" placeholder="课程标题"/>
      </el-form-item>
      <el-form-item label="讲师名称">
        <el-select v-model="searchObj.teacherId" placeholder="请选择讲师">
          <el-option v-for="teacher in teacherList" :key="teacher.id" :value="teacher.name"/>
        </el-select>
      </el-form-item>
      <el-button type="primary" icon="el-icon-search" @click="fetchData">查询</el-button>
      <el-button type="default" @click="resetData">清空</el-button>
    </el-form>
    <!--表格-->
    <el-table v-loading="listLoading" :data="list" element-loading-text="数据加载中"
              border fit highlight-current-row row-class-name="myClassList">
      <el-table-column label="序号" width="70" align="center">
        <template slot-scope="scope">
          {{(page-1)*limit+scope.$index+1}}
        </template>
      </el-table-column>
      <el-table-column label="课程信息" width="470px" align="center">
        <template slot-scope="scope">
          <div class="info">
            <div class="pic">
              <img :src="scope.row.cover" :alt="scope.row.title" width="150px"/>
            </div>
            <div class="title">
              <a href="">{{scope.row.title}}</a>
              <p>{{scope.row.lessonNum}}课时</p>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center">
        <template slot-scope="scope">
          {{scope.row.gmtCreate.substring(0,10)}}
        </template>
      </el-table-column>
      <el-table-column label="发布时间" align="center">
        <template slot-scope="scope">
          {{scope.row.gmtModified.substring(0,10)}}
        </template>
      </el-table-column>
      <el-table-column label="价格" width="100" align="center">
          <template slot-scope="scope">
            {{Number(scope.row.price)===0?'免费':'￥'+scope.row.price.toFixed(2)}}
          </template>
      </el-table-column>
      <el-table-column prop="buyCount" label="付费学员" width="100" align="center">
        <template slot-scope="scope">
          {{scope.row.buyCount}}人
        </template>
      </el-table-column>
      <el-table-column  label="操作" width="150" align="center">
        <template slot-scope="scope">
          <router-link :to="'/edu/course/info/'+scope.row.id">
            <el-button type="text" size="mini" icon="el-icon-edit">编辑课程信息</el-button>
          </router-link>
          <router-link :to="'/edu/course/chapter/'+scope.row.id">
            <el-button type="text" size="mini" icon="el-icon-edit">编辑课程大纲</el-button>
          </router-link>
          <el-button type="text" size="mini" icon="el-icon-delete" @click="deleteCourseById(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <el-pagination :current-page="page" :page-size="limit" :total="total"
    style="padding: 30px 0" text-align="center" layout="total,prev,pager,next,jumper" @current-change="fetchData"/>
  </div>
</template>
<script>
  const baseUrl = process.env.VUE_APP_BASE_API;
  export default {
    data(){
      return {
        listLoading :true,
        list:null,
        total:0,
        page:1,
        limit:5,
        searchObj:{
          title:'',
          teacherId:'',
          subjectParentId:'',
          subjectId:''
        },
        subjectNestedList:[],
        subSubjectList:[],
        teacherList:[]
      }
    },
    created() {
      this.fetchData();
      this.initSubjectList();
      this.initTeacherList();
    },
    methods:{
      fetchData(page=1){
        this.page = page;
        console.log("加载数据列表");
        this.listLoading = true;
        this.$axios({
          url:baseUrl+`admin/edu/course/${this.page}/${this.limit}`,
          method:'post',
          data:this.searchObj
        }).then((response)=>{
          const res = response.data;
          if(res.success){
            this.list = res.data.rows;
            this.total = res.data.total;
          }
          this.listLoading = false;
        }).catch((error)=>{
          console.log(error)
        })
      },
      initSubjectList(){
        this.$axios({
          url:process.env.VUE_APP_BASE_API+'admin/edu/subject',
          method:'get'
        }).then((response)=>{
          let res = response.data;
          if(res.code==20000){
            //debugger;
            this.subjectNestedList = res.data.items
          }
          /*//debugger;
          for (let i = 0; i < this.subjectNestedList.length; i++) {
            if(this.subjectNestedList[i].id==this.courseInfo.subjectParentId){
              this.subSubjectList = this.subjectNestedList[i].children;
            }
          }*/
        }).catch(function (error){
          console.log(error);
        })
      },
      initTeacherList(){
        this.$axios({
          url:baseUrl+'admin/edu/teacher/',
          method:'get'
        }).then((response)=>{
          const res = response.data;
          if(res.success){
            this.teacherList = res.data.items;
          }
        }).catch((error)=>{
          console.log(error)
        })
      },
      subjectLeaveOneChaged(value){
        console.log(value);
        for (let i = 0; i < this.subjectNestedList.length; i++) {
          if(this.subjectNestedList[i].id==value){
            //debugger;
            this.subSubjectList = this.subjectNestedList[i].children
            this.searchObj.subjectId = ''
          }
        }
      },
      resetData(){
        this.searchObj = {};
        this.subSubjectList = [];
        this.fetchData();
      },
      deleteCourseById(id){
          this.$confirm('是否删除','提示',{
            confirmButtonText:'确定',
            cancelButtonText:'取消',
            type:'warning'
          }).then(()=>{
            this.$axios({
              url:baseUrl+'admin/edu/course/'+id,
              method:'delete',
            }).then((resonse)=>{
              if(resonse.data.success==true){
                this.fetchData();
                this.$message({
                  type:'success',
                  message:'删除成功'
                })
              }

            }).catch((response)=>{
              this.$message({
                type:'error',
                message:response.data.message
              })
            })
          }).catch((reponse)=>{
            if(reponse=='cancel'){
              this.$message({
                type:'info',
                message:'已取消删除'
              })
            }else{
              this.$message({
                type:'error',
                message:reponse.message
              })
            }
          })
      }
    }
  }
</script>

