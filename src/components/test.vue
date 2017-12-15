<template>
  <div>
	<button @click="toto = !toto"> 切换到{{toto? '甘特图': 'tree-grid'}}</button>
	<div v-if="toto">
		<tree-grid
			:columns="columns"
			:rowdata="data"
			:needUpdate="needUpdate"
			leafUrl="http://api.fanhua.dev.tianheng-uestc.com/api/v1/apitest//component/treegrid/base?parentid="
			:treeLoading="treeLoading"

			@refreshTable="refreshTable"
			@uploadmodify="uploadmodify"
			@uploaddelete="uploaddelete"
        ></tree-grid>
	</div>
	<div v-if="!toto">
		<gante />
	</div>
  </div>
</template>

<script>
import treeGrid from '@/components/tree-grid'
import gante from '@/components/gante'
export default {
  data(){
    return {
		toto: true,
        columns: [
            {name: 'ID', prop: 'id', width: 50},
            {name: '删除',  delete: true }, //删除、操作不需要width
            {name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true},
            {name: '操作',  operate: true }, //删除、操作不需要width
            // {name: 'level', prop: 'level', width: 120},
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
      treeGrid,gante
  }
}
</script>

