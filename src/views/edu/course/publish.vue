<template>
    <div class="app-container">
        <h2 style="text-align:center;">发布新课程</h2>
        <el-steps :active="3" process-status="wait" align-center style="margin-bottom:40px;">
            <el-step title="填写课程基本信息"/>
            <el-step title="创建课程大纲"/>
            <el-step title="提交审核"/>
        </el-steps>
        <el-form label-width="120px">
            <div class="ccInfo">
                <img :src="coursePulish.cover"/>
                <div class="main">
                  <h2>{{coursePulish.title}}</h2>
                  <p class="gray">共{{coursePulish.lessonNum}}课时</p>
                  <p><span>所属分类：{{coursePulish.subjectLeaveOne}}-{{coursePulish.subjectLeaveTwo}}</span></p>
                  <p>课程讲师：{{coursePulish.teacherName}}</p>
                  <h3 class="red">￥{{coursePulish.price}}</h3>
                </div>
            </div>
            <el-form-item>
                <el-button @click="previous">返回修改</el-button>
                <el-button :disabled="saveBtnDisabled" type="primary" @click="publish">发布课程</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>
<script>
  const baseUrl = process.env.VUE_APP_BASE_API
export default {
    data(){
        return {
            saveBtnDisabled:false,
            courseId:'',
            coursePulish:{}
        }
    },
    created(){
        console.log('publish create');
        this.init();
    },
    methods:{
        previous(){
            console.log('previous')
            this.$router.push({path:'/edu/course/chapter/'+this.courseId});
        },
        publish(){
            this.$axios({
              url:baseUrl+'admin/edu/course/courseInfo/'+this.courseId,
              method:'post',
            }).then(response=>{
              this.$router.push({path:'/edu/course/list'})
            }).catch(response=>{
              console.log(response);
            })

        },
        init(){
          if(this.$route.params&&this.$route.params.id){
            this.courseId = this.$route.params.id;
            //根据id获取课程基本信息
            this.fechCoursePublishById();
          }

        },
      fechCoursePublishById(){
        this.$axios({
          url:baseUrl+'admin/edu/course/coursePubicInfo/'+this.courseId,
          method:'get',
        }).then(response=>{
          this.coursePulish = response.data.data.items;
        }).catch(response=>{
          console.log(response);
        })
      }
    }
}
</script>
<style scoped>
  .ccInfo{
    background: #f5f5f5;
    padding:20px;
    overflow: hidden;
    border:1px dashed #DDDDDD;
    marigin-bottom:40px;
    position: relative;
  }
  .ccInfo img{
    background: #d6d6d6;
    width: 500px;
    height:278px;
    display: block;
    float: left;
    border:none;
  }
  .ccInfo .main{
    margin-left: 520px;
  }

  .ccInfo .main h2{
    font-size: 28px;
    margin-bottom: 30px;
    line-height: 1;
    font-weight: normal;
  }

  .ccInfo .main p{
    margin-bottom:10px;
    word-wrap:break-word;
    line-height: 24px;
    max-height: 48px;
    overflow: hidden;
  }

  .ccInfo .main h3{
    left:540px;
    bottom:20px;
    line-height: 1;
    font-size: 28px;
    color:#d32f24;
    font-weight: normal;
    position: absolute;
  }
</style>
