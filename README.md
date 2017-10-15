tree-grid-by-vue  
-------

![Image text](https://github.com/lky5230/tree-grid-by-vue/blob/master/src/assets/demo.png)

## 样例
<tree-grid   </br>
    :columns="columns"   </br>
    :rowdata="data"   </br>
    :needUpdate="needUpdate"   </br>
    @refreshTable="refreshTable"   </br>
    @uploadmodify="uploadmodify"   </br>
    @uploaddelete="uploaddelete"   </br>
    :treeLoading="treeLoading"   </br>
></tree-grid>   </br>

### 属性

<!--列属性-->   </br>
#1、   </br>
columns: [   </br>
    {name: 'ID', prop: 'id', width: 120},   </br>
    {name: '删除',  delete: true },    </br>
    {name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true},   </br>
    {name: '操作',  operate: true },    </br>
    {name: 'level', prop: 'level', width: 120},   </br>
    {name: 'url', prop: 'url', edit: true},   </br>
]   </br>

<!--行数据的格式-->   </br>
#2、   </br>
rowdata:[{   </br>
    "id": 87, //必须，唯一id。   </br>
    "level": 1, //必须，哪个层级。   </br>
    "parentid": 1, //必须，父层的id，若是最外层就是自己的id。   </br>
    "name": "生产分析",    </br>
    "url": "N/A",   </br>
    "isleaf": 0   </br>
}]

<!--是否接收父组件rowdata传入-->   </br>
#3、   </br>
[[比如：Date.now()]每次父组件需要更新rowdata时，需要给needUpdate传入[时间戳/uuid]等来监听。]   </br>
:needUpdate="needUpdate"    </br>

<!--表格loading状态-->   </br>
#4、   </br>
[布尔值]   </br>
:treeLoading="treeLoading"    </br>

### 事件
#1、   </br>
[该函数是用来刷新表格并重新传入rowdata数据，这里还需要needUpdate。]   </br>
@refreshTable="refreshTable"   </br>
#2、   </br>
[提交修改请求，比如修改行，增加行等，uploadmodify函数的参数是一个数组 [data, successFn_callback, faildFn_callback]，其中data表示修改的对象信息，data的id属性若含有'__timestamp__'字符，表示这是新增的行，successFn_callback和faildFn_callback需要在请求成功或者失败后调用。]   </br>
@uploadmodify="uploadmodify"   </br>
#3、   </br>
[提交删除所选行请求，uploaddelete函数的参数是一个数组 [deleteIdArray, successFn_callback, faildFn_callback]，其中deleteIdArray表示待删除的id集合，successFn_callback和faildFn_callback需要在请求成功或者失败后调用。]   </br>
@uploaddelete="uploaddelete"   </br>
