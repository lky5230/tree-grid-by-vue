<template>
  <div>
    <tree-grid
        :columns="columns"
        :rowdata="data"
        :needUpdate="needUpdate"
        @refreshTable="refreshTable"
        @uploadmodify="uploadmodify"
        @uploaddelete="uploaddelete"
        :treeLoading="treeLoading"
        ></tree-grid>
  </div>
</template>

<script>
import treeGrid from '@/components/tree-grid'
export default {
  data(){
    return {
        columns: [
            {name: 'ID', prop: 'id', width: 120},
            {name: '删除',  delete: true }, //删除、操作不需要width
            {name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true},
            {name: '操作',  operate: true }, //删除、操作不需要width
            {name: 'level', prop: 'level', width: 120},
            {name: 'url', prop: 'url', edit: true},
        ],
        data: [],
        needUpdate: Date.now(),

        treeLoading: false,
    }
  },
  created(){
      this.treeLoading = true;
      this.$http.get('static/menu.json').then(res=>{
            setTimeout(()=>{
                this.treeLoading = false;
                this.data = res.data;
                this.needUpdate = Date.now();
            }, 1400);
        });
  },
  methods: {
      refreshTable(){
        this.treeLoading = true;
        this.$http.get('static/menu.json').then(res=>{
            setTimeout(()=>{
                this.treeLoading = false;
                this.data = res.data;
                this.needUpdate = Date.now();
            }, 1400);
        });
      },
      uploadmodify([data, successFn_callback, faildFn_callback]){
            console.log(data)
            setTimeout(()=>{
                if(Math.random() > 0.1){
                    // 上传成功
                    successFn_callback()
                }else{
                    // 上传失败
                    faildFn_callback()
                }
            }, 1100);
      },
      uploaddelete([deleteIdArray, successFn_callback, faildFn_callback]){
            console.log(deleteIdArray)
            setTimeout(()=>{
                if(Math.random() > 0.1){
                    // 删除成功
                    successFn_callback()
                }else{
                    // 删除失败
                    faildFn_callback()
                }
            }, 1100);
      },

  },
  components: {
      treeGrid
  }
}
</script>

