<template>
   <div class="app-container">
       <el-input v-model="filterText" placeholder="Fileter KeyWord" style="margin-bottom:30px;" />
       <el-tree ref="subjectTree" :data="subjectList" :props="defaultProps"
       :filter-node-method="filterNode" class="filter-tree" default-expand-all=""/>
   </div>
</template>
<script>
export default {
    data(){
        return {
            filterText:'',
            subjectList:[],
            defaultProps:{
                children:'children',
                label:'title'
            }
        }
    },
    watch:{
        filterText(val){
            this.$refs.subjectTree.filter(val);
        }
    },
    created(){
        this.fetchNodeList()
    },
    methods:{
        fetchNodeList(){
            this.$axios({
                url:process.env.VUE_APP_BASE_API+'admin/edu/subject',
                method:'get'
            }).then((response)=>{
                    let res = response.data;
                   if(res.code==20000){
                       this.subjectList = res.data.items
                   }
            }).catch(function (error){
                console.log(error);
            })
        },
        filterNode(value,data){
            if(!value){
                return true;
            }
            return data.title.toLowerCase().indexOf(value.toLowerCase())!==-1
        }
    }
}
</script>