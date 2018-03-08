 <template>
    <div>
        <button @click="clickDel">删除</button>
        <tree-grid
            :columns="columns" [列定义][拥有属性：'name', 'prop', 'width', 'delete', 'isTree', 'edit', 'operate', 'select', 'optionList', 'funcList']
            :rowdata="data" [行定义]数组：[目前每项必须有属性：'isleaf(用于延迟加载)','id','parentid']
            :needUpdate="needUpdate" [用于组件对新数据产生响应]
            :onlyAddOne="true"  [是否只能在已存在的父节点下添加子节点，默认false]
            :onlyLineEdit="true"  [是否只能编辑行，不能增加同级或下级，默认false]
            :showDeleteBtn="false"[是否在删除列表头显示删除按钮，默认false]
            leafUrl="http://api.fanhua.dev.tianheng-uestc.com/api/v1/apitest/component/treegrid/base?parentid="  [叶子节点的异步展开接口，请求时会拼接一个参数，即点击的节点的id]
            /*
            *（*）当存在leafUrl时该属性必须存在！
            *（*）表示取请求结果的permission数组，
            *（*）该数组里id取permission_id字段，name取permission_name字段，checked取select字段
            */
            :funcListAlias="{
                funcList: 'permission',
                id: 'permission_id',
                name: 'permission_name',
                checked: 'select',
            }"
            :treeLoading="treeLoading" [组件的loading]
            @currentDate="currentDate" [监听内部rowdata数据的变动]
            @refreshTable="refreshTable" [重新刷新rowdata]
            @uploadmodify="uploadmodify" [监听修改或新增行]参数：一个数组，[==数据源, 成功时调用的回调函数, 失败时调用的回调函数==]
            @uploaddelete="uploaddelete"> [监听多选框选择的行]参数：一个数组，[==数据源, 成功时调用的回调函数, 失败时调用的回调函数==]
        </tree-grid>
    </div>
</template>

<script>
import treeGrid from '@/commonComponents/tree-grid'
export default {
    data() {
        return {
            columns: [
                { name: 'ID', prop: 'id', width: 50 },
                { name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true, delete: true },
                { name: '操作', operate: true, width: 50 },
                { name: 'url', prop: 'url', edit: true, width: 50 },
                { name: '下拉的', width: 50, prop: 'selectItem', select: true, optionList: [1,2,3,4] },
                /*
                * 功能选择：取得的数据，prop对应字段应该 -> [
                *   {name:'删除', checked: true},
                *   {name:'更新', checked: false},
                *   {name:'查看', checked: false}
                * ]
                */
                { name: '功能选择', prop: 'funcList', funcList: true, width: 50 },
            ],
            data: [],
            needUpdate: Date.now(),

            treeLoading: false,
        }
    },
    created() {
        this.treeLoading = true;
        this.$http.get('static/menu.json').then(res => {
            setTimeout(() => {
                this.treeLoading = false;
                this.data = res.data;
                this.needUpdate = Date.now();
            }, 1400);
        });
    },
    methods: {
        /*
        * 其他：外部触发删除
        */
        clickDel() {
            window.document.querySelector('#tree_grid___id').click();
        },
        /*
        * tree-grid暴露以下事件
        */
        refreshTable() {
            this.treeLoading = true;
            this.$http.get('static/menu.json').then(res => {
                setTimeout(() => {
                    this.treeLoading = false;
                    this.data = res.data;
                    this.needUpdate = Date.now();
                }, 1400);
            });
        },
        uploadmodify([data, successFn_callback, faildFn_callback]) {
            console.log(data)
            setTimeout(() => {
                if (Math.random() > 0.1) {
                    // 上传成功
                    successFn_callback('上传success！')
                } else {
                    // 上传失败
                    faildFn_callback('上传失败！---')
                }
            }, 1100);
        },
        uploaddelete([deleteIdArray, successFn_callback, faildFn_callback]) {
            console.log(deleteIdArray)
            setTimeout(() => {
                if (Math.random() > 0.1) {
                    // 删除成功
                    successFn_callback()
                } else {
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

