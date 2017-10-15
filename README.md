tree-grid-by-vue  
-------

![Image text](https://github.com/lky5230/tree-grid-by-vue/blob/master/src/assets/demo.png)

## 样例
<tree-grid
    :columns="columns"
    :rowdata="data"
    :needUpdate="needUpdate"
    @refreshTable="refreshTable"
    @uploadmodify="uploadmodify"
    @uploaddelete="uploaddelete"
    :treeLoading="treeLoading"
></tree-grid>

### 属性

<!--列属性-->
#1、
columns: [
    {name: 'ID', prop: 'id', width: 120},
    {name: '删除',  delete: true }, 
    {name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true},
    {name: '操作',  operate: true }, 
    {name: 'level', prop: 'level', width: 120},
    {name: 'url', prop: 'url', edit: true},
]

<!--行数据的格式-->
#2、
rowdata:[{
    "id": 87, //必须，唯一id。
    "level": 1, //必须，哪个层级。
    "parentid": 1, //必须，父层的id，若是最外层就是自己的id。

    "name": "生产分析", 
    "url": "N/A",
    "isleaf": 0
}]

<!--是否接收父组件rowdata传入-->
#3、
[[比如：Date.now()]每次父组件需要更新rowdata时，需要给needUpdate传入[时间戳/uuid]等来监听。]
:needUpdate="needUpdate" 

<!--表格loading状态-->
#4、
[布尔值]
:treeLoading="treeLoading" 

### 事件
#1、
[该函数是用来刷新表格并重新传入rowdata数据，这里还需要needUpdate。]
@refreshTable="refreshTable"
#2、
[提交修改请求，比如修改行，增加行等，uploadmodify函数的参数是一个数组 [data, successFn_callback, faildFn_callback]，其中data表示修改的对象信息，data的id属性若含有'__timestamp__'字符，表示这是新增的行，successFn_callback和faildFn_callback需要在请求成功或者失败后调用。]
@uploadmodify="uploadmodify"
#3、
[提交删除所选行请求，uploaddelete函数的参数是一个数组 [deleteIdArray, successFn_callback, faildFn_callback]，其中deleteIdArray表示待删除的id集合，successFn_callback和faildFn_callback需要在请求成功或者失败后调用。]
@uploaddelete="uploaddelete"
