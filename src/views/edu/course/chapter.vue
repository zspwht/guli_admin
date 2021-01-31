<template>
    <div class="app-container">
        <h2 style="text-align:center;">发布新课程</h2>
        <el-steps :active="2" process-status="wait" align-center style="margin-bottom:40px;">
            <el-step title="填写课程基本信息"/>
            <el-step title="创建课程大纲"/>
            <el-step title="提交审核"/>
        </el-steps>
        <el-form label-width="120px">
            <el-form-item>
                <el-button type="text" @click="diaologChapterFormVisible = true">添加章节</el-button>
                <!--章节表单-->
                <el-dialog :visible.sync="diaologChapterFormVisible" title="添加章节">
                  <el-form :model="chapter" label-width="120px">
                      <el-form-item label="章节标题">
                        <el-input v-model="chapter.title"/>
                      </el-form-item>
                    <el-form-item label="章节排序">
                      <el-input v-model="chapter.sort" :min="0" controls-position="right"/>
                    </el-form-item>
                  </el-form>
                  <div slot="footer" class="dialog-footer">
                      <el-button @click="diaologChapterFormVisible=false">取消</el-button>
                      <el-button type="primary" @click="saveOrUpdateChapter">确定</el-button>
                  </div>
                </el-dialog>
                <!--课时表单-->
                <el-dialog :visible.sync="diaologVideoFormVisible" title="添加课时">
                  <el-form :model="video" label-width="120px">
                    <el-form-item label="课时标题">
                      <el-input v-model="video.title"/>
                    </el-form-item>
                    <el-form-item label="课时排序">
                      <el-input v-model="video.sort" :min="0" controls-position="right"/>
                    </el-form-item>
                    <el-form-item label="是否免费">
                      <el-radio-group v-model="video.isFree">
                        <el-radio :label="0">默认</el-radio>
                        <el-radio :label="1">免费</el-radio>
                      </el-radio-group>
                    </el-form-item>
                    <el-form-item label="上传视频">
                        <!--todo-->
                    </el-form-item>
                  </el-form>
                  <div slot="footer" class="dialog-footer">
                    <el-button @click="diaologVideoFormVisible=false">取消</el-button>
                    <el-button :disabled="saveBtnVideoDisabled" type="primary" @click="saveOrUpdateVideo">确定</el-button>
                  </div>
                </el-dialog>
                  <!--章节 -->
                <ul class="chapterList">
                  <li v-for="chapter in chapterNestedList" :key="chapter.id">
                    <p>
                      {{chapter.title}}
                      <span class="acts">
                        <el-button type="text" @click="diaologVideoFormVisible = true,chapterId = chapter.id">添加课时</el-button>
                        <el-button style="" type="text" @click="editChapter(chapter.id)">编辑</el-button>
                        <el-button type="text" @click="deleteChapter(chapter.id)">删除</el-button>
                      </span>
                    </p>

                    <!--视频 -->
                    <ul class="chapterList videoList">
                      <li v-for="video in chapter.children" :key="video.id">
                        <p>
                          {{video.title}}
                          <span class="atcs">
                            <el-button type="text" @click="editVideo(video.id)">编辑</el-button>
                            <el-button type="text" @click="deleteVideo(video.id)">删除</el-button>
                          </span>
                        </p>
                      </li>
                    </ul>
                  </li>
                </ul>
                <el-button @click="previous">上一步</el-button>
                <el-button :disabled="saveBtnDisabled" type="primary" @click="next">保存并下一步</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>
<script>
const BaseUrl = process.env.VUE_APP_BASE_API;
export default {
    data(){
        return {
            saveBtnDisabled:false,
            courseId:'',
            chapterNestedList:[],
            chapter:{ //章节对象
              title:'',
              sort:0
            },
            diaologChapterFormVisible:false, //是否显示章节表单
            //课时数据
            saveBtnVideoDisabled:false, //课时按钮是否禁用
            diaologVideoFormVisible:false, //是否显示课时表单，
            chapterId:'',
            video:{
              title:'',
              sort:0,
              isFree: 0,
              videoSourceId:''
            }
        }
    },
    created(){
        console.log('chapter create');
        this.init();
    },
    methods:{
        previous(){
            console.log('previous')
            this.$router.push({path:'/edu/course/info/'+this.courseId});
        },
        next() {
          console.log('next')
          this.$router.push({path: '/edu/course/publish/'+this.courseId})
        },
        init(){
          //debugger;
          if(this.$route.params&&this.$route.params.id){
            this.courseId = this.$route.params.id;
            //根据id获取课程基本信息
            this.fetchChapterNestedListByCourseId();
          }
        },
      fetchChapterNestedListByCourseId(){
          this.$axios({
            url:BaseUrl+'admin/edu/chapter/nestedList/'+this.courseId,
            method:'get',
          }).then(response=>{
            this.chapterNestedList = response.data.data.items;
          }).catch(error=>{
            console.log(error);
          })
      },
      //新增或则更新章节
      saveOrUpdateChapter(){
          //debugger;
        this.diaologChapterFormVisible = true;
        if(!this.chapter.id){
          this.saveChapter();
        }else{
          this.updateChapter();
        }
      },
      saveChapter(){
          this.chapter.courseId = this.courseId;
          this.$axios({
            url:BaseUrl+"admin/edu/chapter/saveChapter",
            method:'post',
            data:this.chapter
          }).then(response=>{
            let res = response.data;
            if(res.success==true){
              this.$message({
                type:"success",
                message:"保存成功"
              })
              this.helpSave();
            }
          }).catch(error=>{
            console.log(error)
          })
      },
      updateChapter(){
        this.$axios({
          url:BaseUrl+`admin/edu/chapter/${this.chapter.id}`,
          method:'post',
          data:this.chapter
        }).then(reponse=>{
          let res = reponse.data;
          this.$message({
            type:'success',
            message:'修改成功'
          })
          this.helpSave();
        }).catch(error=>{
          console.log(error);
          this.$message({
            type:"error",
            message:'修改失败'
          })
        })
      },
      editChapter(chapterId){
        this.diaologChapterFormVisible = true;
        this.$axios({
          url:BaseUrl+`admin/edu/chapter/${chapterId}`,
          method:'get'
        }).then(reponse=>{
          let res = reponse.data;
          this.chapter = res.data.item;
        }).catch(error=>{
          console.log(error);
        })
      },
      helpSave(){
        this.diaologChapterFormVisible = false;
        this.fetchChapterNestedListByCourseId();
        this.chapter.title = '';
        this.chapter.sort = 0;
        this.saveBtnDisabled = false;
      },
      deleteChapter(chapterId){
          this.$confirm('此操作将永久删除该记录，是否继续','提示',{
            confirmButtonText:'确定',
            cancelButtonText:'取消',
            type:'warning'
          }).then(()=>{
            this.$axios({
              url:BaseUrl+`admin/edu/chapter/${chapterId}`,
              method:'delete'
            }).then(reponse=>{
              let res = reponse.data;
              if(res.success==true){
                this.fetchChapterNestedListByCourseId();
                this.$message({
                  type:'success',
                  message:'删除成功',
                })
              }
            }).catch(error=>{
              console.log(error);
              this.$message({
                type:'error',
                message:error.data.message
              })
            })
          }).catch(response=>{
            if(response=='cancel'){
              this.$message({
                type:'info',
                message:'已取消删除'
              })
            }else{
              this.$message({
                type:'error',
                message:response.message
              })
            }
          })
      },
      //处理课时视频

      saveOrUpdateVideo(){
        this.saveBtnVideoDisabled = true;
        if(!this.video.id){
          this.saveVideo();
        }else{
          this.updateVideo();
        }
      },
      saveVideo(){
        this.video.courseId = this.courseId;
        this.video.chapterId = this.chapterId;
        this.$axios({
          url:BaseUrl+"admin/edu/video/saveVideo",
          method:'post',
          data:this.video
        }).then(response=>{
          let res = response.data;
          if(res.success==true){
            this.$message({
              type:"success",
              message:"保存成功"
            })
            this.helpSaveVideo();
          }
        }).catch(error=>{
          console.log(error)
        })
      },
      updateVideo(){
        this.$axios({
          url:BaseUrl+`admin/edu/video/${this.video.id}`,
          method:'post',
          data:this.video
        }).then(reponse=>{
          let res = reponse.data;
          this.$message({
            type:'success',
            message:'修改成功'
          })
          this.helpSaveVideo();
        }).catch(error=>{
          console.log(error);
          this.$message({
            type:"error",
            message:'修改失败'
          })
        })
      },
      editVideo(videoId){
        this.diaologVideoFormVisible = true;
        this.$axios({
          url:BaseUrl+`admin/edu/video/${videoId}`,
          method:'get'
        }).then(reponse=>{
          let res = reponse.data;
          this.video = res.data.item;
        }).catch(error=>{
          console.log(error);
        })
      },
      helpSaveVideo(){
        this.diaologVideoFormVisible = false;
        this.fetchChapterNestedListByCourseId();
        this.video.title = '';
        this.video.sort = 0;
        this.video.videoSourceId = '';
        this.saveBtnVideoDisabled = false;
      },
      deleteVideo(videoId){
        this.$confirm('此操作将永久删除该记录，是否继续','提示',{
          confirmButtonText:'确定',
          cancelButtonText:'取消',
          type:'warning'
        }).then(()=>{
          this.$axios({
            url:BaseUrl+`admin/edu/video/${videoId}`,
            method:'delete'
          }).then(reponse=>{
            let res = reponse.data;
            if(res.success==true){
              this.fetchChapterNestedListByCourseId();
              this.$message({
                type:'success',
                message:'删除成功',
              })
            }
          }).catch(error=>{
            console.log(error);
            this.$message({
              type:'error',
              message:error.data.message
            })
          })
        }).catch(response=>{
          if(response=='cancel'){
            this.$message({
              type:'info',
              message:'已取消删除'
            })
          }else{
            this.$message({
              type:'error',
              message:response.message
            })
          }
        })
      }
    }
}
</script>
<style scoped>
  .chapterList{
    position: relative;
    list-style: none;
    margin:0;
    padding:0;
  }
  .chapterList li{
    position: relative;
  }
  .chapterList p{
    float: left;
    font-size: 20px;
    margin:10px 0px;
    padding:10px;
    height: 70px;
    line-height: 50px;
    width: 100%;
    border: 1px solid #dddddd;
  }
  .chapterList acts{
    float: right;
    font-size: 14px;
  }
  .videoList{
    padding-left: 50px;
  }
  .videoList p{
    float: left;
    font-size: 14px;
    margin: 10px 0;
    padding: 10px;
    height: 50px;
    line-height: 30px;
    width: 100%;
    border: 1px dotted #dddddd;
  }
</style>
