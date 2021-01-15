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
            <el-form-item label="总课时">
                <el-input-number :min="0" v-model="courseInfo.lessonNum" controls-position="right" placeholder="请填写课程的总课时数"/>
            </el-form-item>
            <!--课程简介 todo -->
            <!--课程封面 todo -->
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
    cover:'',
    price:0
}
export default {
    data(){
        return {
            courseInfo:defaultForm,
            saveBtnDisabled:false,
            subjectNestedList:[], //一级分类
            subSubjectList:[]  //二级分类
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
                this.updataData();
            }
            this.$route.push({path:'/edu/course/chapter/1'});
        },
        init(){
            if(this.$route.params&&this.$route.params.id){
                const id = this.$route.params.id;
                console.log(id);
            }else{
                this.courseInfo = {...defaultForm}
            }
            //初始化分类列表
            this.initSubjectList();
        },
        savaData(){
            this.$axios({
                url:process.env.VUE_APP_BASE_API+'admin/edu/course/saveCourseInfo',
                method:'post'
            }).then((response)=>{
                const res = response.data;
                if(res.success){
                    this.$message({
                        type:'success',
                        message:'保存成功！'
                    });
                }
            }).then((response=>{
                this.$router.push({path:'/edu/course/chapter'+response.data.data.courseId})
            })).catch(error=>{
                console.log(error);
            })
        },
        updataData(){
            this.$router.push({path:'/edu/course/chapter/1'})
        },
        initSubjectList(){
             this.$axios({
                url:process.env.VUE_APP_BASE_API+'admin/edu/subject',
                method:'get'
            }).then((response)=>{
                    let res = response.data;
                   if(res.code==20000){
                       this.subjectNestedList = res.data.items
                   }
            }).catch(function (error){
                console.log(error);
            })
        },
        subjectLeaveOneChanged(value){
            console.log(value);
            for (let i = 0; i < this.subjectNestedList.length; i++) {
               if(this.subjectNestedList[i].id==value){
                   this.subSubjectList = this.subjectNestedList[i].children
                   this.courseInfo.subjectId = ''
               }
            }
        }
    }
}
</script>