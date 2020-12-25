<template>
    <div class="app-container">
        <el-form label-width="120px">
            <el-form-item label="讲师名称">
                <el-input v-model="teacher.name"/>
            </el-form-item>
            <el-form-item label="讲师排序">
                <el-input v-model="teacher.sort" controls-position="right" min="0"/>
            </el-form-item>
            <el-form-item label="讲师头衔">
                <el-select v-model="teacher.level" clearable placeholder="请选择">
                    <el-option :value="1" label="高级讲师"/>
                    <el-option :value="2" label="首席讲师"/>
                </el-select>
            </el-form-item>
            <el-form-item label="讲师资历">
                <el-input v-model="teacher.career" />
            </el-form-item>
            <el-form-item label="讲师简介">
                <el-input v-model="teacher.intro" :rows="10" type="textarea"/>
            </el-form-item>
            <el-form-item>
                <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>
<script>
export default {
    data(){
        return {
            teacher:{
                name:'',
                sort:'',
                level:'',
                career:'',
                intro:''
            },
            saveBtnDisabled:false
        }
    },
    created(){
        debugger;
       if(this.$route.params&&this.$route.params.id){
            const id = this.$route.params.id;
            alert(id);
            this.fechDataByid(id);
       }
    },
    methods:{
        saveOrUpdate(){
            this.savaBtnDisabled = true;
            debugger;
            alert(this.teacher);
            if(!this.teacher.id){
                  this.saveDate();  
            }else{
                this.updataDate(this.teacher.id);
            }
            
        },
        saveDate(){
            this.$axios({
                url:`http://localhost:8885/admin/edu/teacher/add`,
                method:'post',
                data:this.teacher
            }).then((response)=>{
                let res = response.data;
                if(res.code==20000){
                    this.$message({
                        type:'success',
                        message:res.message
                    })
                    this.$router.push({path:'/edu/teacher'});
                }
            }).catch(response=>{
                this.$message({
                    type:'error',
                    message:'保存失败'
                })
            })
        },
        fechDataByid(id){
            this.$axios({
                url:`http://localhost:8885/admin/edu/teacher/${id}`,
                method:'get',
            }).then(response=>{
                let res = response.data;
                if(res.code==20000){
                    console.log(this.teacher);
                    this.teacher = res.data.row;
                    console.log(this.teacher);
                }
            }).catch(response=>{
                this.$message({
                    type:'error',
                    message:'数据获取失败'
                })
            })
        },
        //更新
        updataDate(id){
            this.$axios({
                url:`http://localhost:8885/admin/edu/teacher/${id}`,
                method:'put',
                data:this.teacher
            }).then(response=>{
                let res = response.data;
                if(res.code==20000){
                   this.$message(
                       {
                           type:'success',
                           message:'更新成功'
                       }
                   );
                   this.$router.push({path:'/edu/teacher'});
                }
            }).catch(response=>{
                this.$message({
                    type:'error',
                    message:'更新失败'
                })
            })
        }
    }
}
</script>