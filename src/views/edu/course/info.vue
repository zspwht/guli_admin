<template>
    <div class="app-container">
        <h2 style="text-align:center;">发布新课程</h2>
        <el-steps :active="1" process-status="wait" align-center style="margin-bottom:40px;">
            <el-step title="填写课程基本信息"/>
            <el-step title="创建课程大纲"/>
            <el-step title="提交审核"/>
        </el-steps>
        <el-form label-width="120px">
            <el-form-item label="课程标题">
                <el-input v-model="courseInfo.title" placeholder="示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写"/>
            </el-form-item>
            <!--所属分类 todo -->
            <!-- 一级分类 -->
            <el-form-item label="课程类别">
                <el-select v-model="courseInfo.subjectParentId" placeholder="请选择" @change="subjectLeaveOneChanged">
                    <el-option v-for="subject in subjectNestedList" :key="subject.id" :label="subject.title" :value="subject.id"/>
                </el-select>
                <!--二级分类 -->
                <el-select v-model="courseInfo.subjectId" placeholder="请选择">
                    <el-option v-for="subject in subSubjectList" :key="subject.id" :label="subject.title" :value="subject.id"/>
                </el-select>
            </el-form-item>
            <!--课程讲师 todo -->
            <el-form-item label="课程讲师">
                <el-select v-model="courseInfo.teacherId" placeholder="请选择">
                    <el-option v-for="teacher in teacherList" :key="teacher.id" :label="teacher.name" :value="teacher.id"/>
                </el-select>
            </el-form-item>
            <el-form-item label="总课时">
                <el-input-number :min="0" v-model="courseInfo.lessonNum" controls-position="right" placeholder="请填写课程的总课时数"/>
            </el-form-item>
            <!--课程简介 todo -->
            <el-form-item label='课程简介'>
                <tinymce :heigth='300' v-model="courseInfo.description"></tinymce>
            </el-form-item>
            <!--课程封面 todo -->
            <el-form-item label="课程封面">
                <el-upload :show-file-list="false" :on-success="handleAvatarSuccess"
                :before-upload="beforeAdatarUpload" :action="OSS_PATH+'/admin/oss/file/upload'"
                class="avatar-uploader">
                  <img height="200px" width="200px" :src="courseInfo.cover"/>
                </el-upload>
            </el-form-item>
             <el-form-item label="课程价格">
                <el-input-number :min="0" v-model="courseInfo.price" controls-position="right" placeholder="免费课程请设置为0元"/>
            </el-form-item>
            <el-form-item>
                <el-button :disabled="saveBtnDisabled" type="primary" @click="next">保存并下一步</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>
<script>
const defaultForm = {
    title:'',
    subjectId:'',
    teacherId:'',
    lessonNum:0,
    description:'',
    cover:process.env.VUE_APP_OSSPATH+"/cover/defaultCover.jpg",
    price:0,
    subjectParentId:''
}
import Tinymce from '@/components/Tinymce'
export default {
     components:{Tinymce},
    data(){
        return {
            courseInfo:defaultForm,
            saveBtnDisabled:false,
            subjectNestedList:[], //一级分类
            subSubjectList:[],  //二级分类
            teacherList:[] , //讲师列表
            OSS_PATH:process.env.VUE_APP_OSS,
        }
    },
    watch:{
        $route(to,from){
            console.log('watch $route');
            this.init();
        }
    },
    created(){
        console.log('info create');
        this.init();
    },
    methods:{
        next(){
            console.log('next')
            this.saveBtnDisabled=true;
            if(!this.courseInfo.id){
                this.savaData();
            }else{
                this.updateData();
            }
            //this.$router.push({path:'/edu/course/chapter/1'});
        },
        init(){
            if(this.$route.params&&this.$route.params.id){
                //debugger;
                const id = this.$route.params.id;
                console.log(id);
                //根据id获取课程信息
                this.fetchCourseInfoById(id);
            }else{
                this.courseInfo = {...defaultForm}
              //初始化分类列表
              this.initSubjectList();
              //获取讲师列表
              this.initTeacherList();
            }

        },
        savaData(){
            this.$axios({
                url:process.env.VUE_APP_BASE_API+'admin/edu/course/saveCourseInfo',
                method:'post',
                data:this.courseInfo
            }).then((response)=>{
                //debugger;
                const res = response.data;
                if(res.success){
                    this.$message({
                        type:'success',
                        message:'保存成功！'
                    });
                }
              this.$router.push({path:'/edu/course/chapter/'+response.data.data.courseId})
            })/*.then((response)=>{
                debugger
                this.$router.push({path:'/edu/course/chapter/'+response.data.data.courseId})
            })*/.catch(error=>{
                console.log(error);
            })
        },
        updateData(){
          this.$axios({
            url:process.env.VUE_APP_BASE_API+`admin/edu/course/courseInfo/${this.courseInfo.id}`,
            method:'post',
            data:this.courseInfo
          }).then((response)=>{
            const res = response.data;
            if(res.success){
              this.$message({
                type:'success',
                message:'保存成功！'
              });
              this.$router.push({path:'/edu/course/chapter/'+response.data.data.courseId})
            }
          })/*.then((response=>{
            this.$router.push({path:'/edu/course/chapter'+response.data.data.courseId})
          }))*/.catch(error=>{
            console.log(error);
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
                   //debugger;
               for (let i = 0; i < this.subjectNestedList.length; i++) {
                 if(this.subjectNestedList[i].id==this.courseInfo.subjectParentId){
                   this.subSubjectList = this.subjectNestedList[i].children;
                 }
               }
            }).catch(function (error){
                console.log(error);
            })
        },
        subjectLeaveOneChanged(value){
          //debugger;
            console.log(value);
            for (let i = 0; i < this.subjectNestedList.length; i++) {
               if(this.subjectNestedList[i].id==value){
                 //debugger;
                   this.subSubjectList = this.subjectNestedList[i].children
                   this.courseInfo.subjectId = ''
               }
            }
        },
        //获取讲师列表
        initTeacherList(){
            this.$axios({
                url:process.env.VUE_APP_BASE_API+'admin/edu/teacher/',
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
      //上传图片前
      beforeAdatarUpload(file){
          const isJPG = file.type=="image/jpeg";
          console.log("图片格式"+isJPG);
          const isLt2M= file.size/1024/1024<2;
          console.log("是否大于2M"+isLt2M);
          if(!isJPG){
            this.$message.error("上传头像图片只能是jpg格式!")
          }
          if(!isLt2M){
            this.$message.error("上传图像图片大小不能超过2M!")
          }
          return isJPG&&isLt2M
      },
      //上传成功后
      handleAvatarSuccess(res,file){
        console.log(res);
        console.log(URL.createObjectURL(file.raw)); //base64编码
        this.courseInfo.cover = res.data.uploadUrl;
      },

      //根据id获取课程信息
      fetchCourseInfoById(id){
        this.$axios({
          url:process.env.VUE_APP_BASE_API+`admin/edu/course/courseInfo/${id}`,
          method:'get',
        }).then(response=>{
          this.courseInfo = response.data.data.item;
          this.initSubjectList();
          this.initTeacherList();
        }).catch(response=>{
          this.$message({
            type:'error',
            message:response.message
          })
        })
      }
    }
}
</script>
<style scoped>
    .tinymce-container{
        line-height: 29px;
    }
</style>
