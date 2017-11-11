<tree-grid 
            :columns="columns" 
            :rowdata="data" 
            :needUpdate="needUpdate" 
            :onlyAddOne="true"  //是否只能在存在的节点下增加下级
            :leafUrl="leafUrl"  //异步获取子节点，以点击节点的id为get方法的参数
            :treeLoading="treeLoading"
            @currentDate="currentDate"  //监听每次树的变动
            @refreshTable="refreshTable"   
            @uploadmodify="uploadmodify" 
            @uploaddelete="uploaddelete">
        </tree-grid>
