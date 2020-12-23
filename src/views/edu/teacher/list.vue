<template>
    <div class="app-container">
        讲师列表
        <!--表格 -->
        <el-table v-loading='listLoading' 
        :data="list" 
        element-loading-text="数据加载中"
        border fit highlight-current-row>
            <el-table-column label="序号" width="70" align="center">
                <template slot-scope="scope">{{(page-1)*limit+scope.$index+1}}</template>
            </el-table-column>
            <el-table-column prop="name" label="名称" width="80"/>
            <el-table-column label="头衔" width="80">
                <template slot-scope="scope">
                    {{scope.row.level==1?'高级讲师':'首席讲师'}}
                </template>
            </el-table-column>
            <el-table-column prop="info" label="资历"/>
            <el-table-column prop="gmtCreate" label="添加时间" width="160"/>
            <el-table-column prop="sort" label="排序" width="60"/>
            <el-table-column label="操作" width="200" align="center">
                <template slot-scope="scope">
                    <router-link :to="'/edu/teacher/edit'+scope.row.id">
                        <el-button type="primary" size="mini" icon="el-icon-edit">修改</el-button>
                    </router-link>
                    <el-button type="danger" size="mini" icon="el-icon-delete"
                     @click="removeDataById(scope.row.id)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
    
    
</template>
<script>
export default {
    data(){
        return {
            listLoading:true,
            list:null,
            total:0,
            page:1,
            limit:3,
            searchObj:{}
        }
    },
    created(){
        this.fetchData();
    },
    methods:{
        fetchData(page=1){
            console.log('加载列表');
            debugger;
            this.page = page;
            this.listLoading = true;
           this.$axios({
               url:'http://localhost:8885/admin/edu/teacher/1/3',
               methods:'get',
           }).then((res)=>{
                        debugger;
                        console.log(res.data.data.rows);
                        this.list = res.data.data.rows;
                        this.total = res.data.data.total;
                        this.listLoading = false;
           }).catch(function(error){
               console.log(error);
           })
        }
    }
}
</script>