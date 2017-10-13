<template>
  <div class="tree">
    <!--表头-->
    <ul class="table-header">
        <li 
            class="nowrap"
            v-for="(col, colIndex) in column" 
            :key="colIndex"
            :style="{
                width: col.width? col.width+'px': 'auto',
                flex: col.width? 'none': 1,
            }">
            <span v-if="col.delete == true">
                <button :disabled="deleteBtnDisable" @click="removeLine">
                    <i v-if="deleteBtnDisable" class="fa fa-spin fa-spinner" aria-hidden="true"></i>
                    {{deleteBtnDisable? '删除中...': '删除'}}
                </button>
            </span>
            <span v-else>{{col.name}}</span>
        </li>
    </ul>
    <!--表体-->
    <ul class="table-body">
        <div 
            v-for="(rowItem, rowIndex) in data" 
            @mouseleave="rowMouseLeave(rowItem.id)"
            @mouseenter="rowMouseEnter(rowItem.id)"
            class="table-body-li-wrap"
            v-show="rowItem.show"
            :key="rowItem.id">
            <div v-if="uploading['uploading'+rowItem.id] == true" class="uploading">
                <i class="fa fa-spin fa-spinner" aria-hidden="true"></i>
            </div>
            <div style="display: flex;">
                <li 
                    v-for="(col, colIndex) in column" 
                    class="table-body-li nowrap"
                    :class="{modify: rowItem.modify == true}"
                    :key="colIndex"
                    :style="{
                        width: col.width? col.width+'px': '',
                        flex: col.width? 'none': 1,
                        'padding-left': col.isTree? 24 * rowItem.level+14+'px': '14px'
                    }">
                    <!--树的伸缩按钮-->
                    <template v-if="col.isTree && rowItem.children.length != 0">
                        <span 
                            @click="expand(rowItem, true)" 
                            v-show="rowItem.icon == false">
                            <i class="fa fa-plus-square" aria-hidden="true"></i>
                        </span>
                        <span 
                            @click="expand(rowItem, false)" 
                            v-show="rowItem.icon == true">
                            <i class="fa fa-minus-square" aria-hidden="true"></i>
                        </span>  
                    </template>
                    <!--不可编辑-->
                    <span v-if="!col.edit"> {{rowItem[col.prop]}} </span>
                    <!--可编辑-->
                    <template v-if="col.edit">
                        <span 
                            style=" text-align: left;"
                            :title="rowItem[col.prop]" 
                            v-show="!operateStatus['status'+rowItem.id]" 
                            class="no-wrap">
                            {{rowItem[col.prop]}}
                        </span>
                        <input 
                            v-show="operateStatus['status'+rowItem.id]" 
                            type="text"
                            @input="inputEdit(rowItem.id, col.prop, $event)"
                            :value="lineValue['lineValue'+col.prop+rowItem.id]" />
                    </template>
                    
                    <!--操作按钮-->
                    <template v-if="col.operate">
                        <span v-show="!operateStatus['status'+rowItem.id]">
                            <button @click="operateBtn(rowItem.id)">
                                编辑
                            </button>
                            <button @click="addLine(rowItem.id, 1)">
                                增加同级
                            </button>
                            <button @click="addLine(rowItem.id, 2)">
                                增加下级
                            </button>
                            <button v-if="rowItem.modify" @click="upload(rowItem.id)">
                                上传
                            </button>
                        </span>
                        <span v-show="!!operateStatus['status'+rowItem.id]">
                            <button @click="saveBtn(rowItem.id)">
                                确认
                            </button>
                            <button @click="cancel(rowItem.id)">
                                撤销
                            </button>
                        </span>
                    </template>
                    <!--删除-复选框-->
                    <template v-if="col.delete">
                        <span 
                        @click="checkbox_change(rowItem.id)" 
                        class="checkbox-span"
                        :class="{active: checkboxControl['active'+rowItem.id]}">
                            <i v-if="checkboxControl['active'+rowItem.id]" class="fa fa-check-circle" aria-hidden="true"></i>
                            <i v-if="!checkboxControl['active'+rowItem.id]" class="fa fa-circle-o" aria-hidden="true"></i>
                        </span>
                    </template>
                </li>
            </div>
        </div>
    </ul>
  </div>    
</template>

<script>

export default {
    name: 'treeGrid', 
    data() {
        return {
            column: [
                {name: 'ID', prop: 'id', width: 120},
                {name: 'name字段', prop: 'name', width: 260, isTree: true, edit: true},
                {name: '操作', width: 250, operate: true },
                {name: '删除', width: 60, delete: true },
                {name: 'level', prop: 'level', width: 120},
                {name: 'url', prop: 'url', edit: true},
            ],
            data:[],
            // 每行数据对象的格式
            data_format: null,
            // 移入移出-控制
            operate:{},
            // 每行的编辑状态
            operateStatus: {},
            // 每行的上传loading
            uploading: {},
            // 控制输入框的值
            lineValue:{},
            // 复选框控制
            checkboxControl: {},
            // 待删除id集合
            deleteIdArr: [],
            //表头-删除btn
            deleteBtnDisable: false,
        };
    },
    
    created(){
        this.$http.get('static/menu.json').then(res=>{
            this.init(res)
        });
        
    },
    watch: {
        
    },
    methods: {
        init(res){
            // 整理data
            this.data = combineData(cleanData(res.data));
            // 得到data格式
            let data_format = {...this.data[0]};
            for(let key in data_format){
                if(data_format.hasOwnProperty(key)){
                    data_format[key] = '';
                }
            };
            this.data_format = data_format;
            // 初始化
            this.initValue();
        },
        //初始化
        initValue(){
            let row = [...this.data];
            let col = [...this.column];
            for(let i=0; i <row.length; i++){
                // 初始化checkbox
                this.$set(this.checkboxControl, 'active'+row[i].id, false);
                for(let j=0; j<col.length; j++){
                    if(col[j].edit == true){
                        // 初始化单元格input
                        this.$set(this.lineValue, 'lineValue'+col[j].prop+row[i].id, row[i][col[j].prop]);
                    }
                }
            }
        },
        //行的移入、移出
        rowMouseEnter(id){
            this.$set(this.operate, 'operate'+id, true);
        },
        rowMouseLeave(id){
            this.$set(this.operate, 'operate'+id, false);
        },
        //行的伸缩
        expand(rowItem, bool){
            let d = [...this.data];
            let lastLine = null;
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItem.id){
                    d[i].icon = bool;
                    let temp_level = Number.MAX_SAFE_INTEGER;
                    let k = i;
                    let clickLevel = d[i].level;
                    while(123456){
                        k++;
                        lastLine = k;
                        if(d[k] == undefined) return [(k-1), d];
                        if(bool == false){
                            if(d[k].level > clickLevel){
                                d[k].show = false;
                            }else{
                                break
                            }
                        }else{
                            if(d[k].level > clickLevel){
                                if(d[k].level > temp_level){
                                    continue ;
                                }else if(d[k].level == temp_level){
                                    temp_level = Number.MAX_SAFE_INTEGER;
                                }
                                if(d[k].icon == false){
                                    temp_level = d[k].level;
                                }
                                d[k].show = true;
                            }else{
                                break;
                            }
                        }
                    }
                }
            };
            this.data = d;
            return [lastLine, d];
        },
        //行的输入事件
        inputEdit(rowItemId, colProp, e){
            let d = [...this.data];
            let col = [...this.column];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    d[i].editing = true;
                    break;
                }
            };
            this.data = d;
            this.$set(this.lineValue, 'lineValue'+colProp+rowItemId, e.target.value);
        },
        //btn-编辑
        operateBtn(rowItemId){
            //打开编辑状态
            this.$set(this.operateStatus, 'status'+rowItemId, true)
        },
        //btn-保存
        saveBtn(rowItemId){
            let d = [...this.data];
            let col = [...this.column];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    if(d[i].editing == true){
                        d[i].modify = true;
                    }
                    for(let j=0; j<col.length; j++){
                        if(col[j].edit == true){
                            d[i][col[j].prop] = this.lineValue['lineValue'+col[j].prop+rowItemId];
                        }
                    };
                    break;
                }
            };
            this.data = d;
            //关闭编辑状态
            this.$set(this.operateStatus, 'status'+rowItemId, false)
        },
        //btn-撤销
        cancel(rowItemId){
            let d = [...this.data];
            let col = [...this.column];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    for(let j=0; j<col.length; j++){
                        if(col[j].edit == true){
                            this.lineValue['lineValue'+col[j].prop+rowItemId] = d[i][col[j].prop]
                        }
                    };
                    break;
                }
            };
            //关闭编辑状态
            this.$set(this.operateStatus, 'status'+rowItemId, false)
        },
        //btn-上传
        upload(rowItemId){
            let d = [...this.data];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    // 显示loading
                    this.$set(this.uploading, 'uploading'+rowItemId, true);
                    
                    setTimeout(()=>{
                        if(Math.random() > 0.6){
                            // 上传成功
                            success.call(this)
                        }else{
                            // 上传失败
                            faild.call(this)
                        }
                    }, 400);
                    
                }
            }
            function success(){
                let d = [...this.data];
                let col = [...this.column];
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        d[i].modify = false;
                        d[i].editing = false;
                        break;
                    }
                };
                this.data = d;
                this.$set(this.uploading, 'uploading'+rowItemId, false);
            }
            function faild(){
                console.log('失败')
                this.$set(this.uploading, 'uploading'+rowItemId, false);
            }
        },
        //btn-增加
        addLine(rowItemId, indexed){
            if(indexed == 1){
                // 1、增加同级
                let d = [...this.data];
                let rowItem = null;
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        rowItem = d[i];
                    };
                };
                let [lastLine, nowData] = this.expand(rowItem, false);
                let newId = Date.now();
                nowData.splice(lastLine, 0, {...this.data_format, ...{
                    id: newId,
                    level: rowItem.level,
                    parentid: rowItem.parentid,
                    
                    modify: true,
                    show: true,
                    
                    //children: [],//长度为0，就没有+/-
                }});
                
                //打开编辑状态
                this.$set(this.operateStatus, 'status'+newId, true)
                this.data = nowData;
            }else if(indexed == 2){
                // 2、增加下级

            }
        },
        //复选框切换
        checkbox_change(rowItemId){
            let old_bool = this.checkboxControl['active'+rowItemId];
            this.$set(this.checkboxControl, 'active'+rowItemId, !old_bool);
            let now_bool = !old_bool;
            if(now_bool){
                // 选中的规则
                let d = [...this.data];
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        let k = i;
                        let clickLevel = d[i].level;
                        while(123456){
                            k++;
                            if(d[k] == undefined) return;
                            if(d[k].level > clickLevel){
                                this.$set(this.checkboxControl, 'active'+d[k].id, true);
                            }else{
                                break;
                            }
                        };
                        break;
                    }
                }
            }else{
                // 不选中的规则
                let d = [...this.data];
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        let k = i;
                        let clickLevel = d[i].level;
                        this.$set(this.checkboxControl, 'active'+d[i].parentid, false);
                        while(123456){
                            k++;
                            if(d[k] == undefined) return;
                            if(d[k].level > clickLevel){
                                this.$set(this.checkboxControl, 'active'+d[k].id, false);
                            }else{
                                break;
                            }
                        };
                        repeatFn.call(this, d[i]);
                        function repeatFn(item){
                            this.$set(this.checkboxControl, 'active'+item.parentid, false);
                            if(item.parentid == item.id){
                                return ;
                            }
                            for(let i=0; i<d.length; i++){
                                if(d[i].id == item.parentid){
                                    repeatFn.call(this, d[i]);
                                }
                            };
                        };
                        break;
                    }
                }
            };
            
            //更新待删除集合：deleteIdArr
            this.deleteIdArr = [];
            for(let key in this.checkboxControl){
                if(this.checkboxControl.hasOwnProperty(key)){
                    if(this.checkboxControl[key] == true){
                        this.deleteIdArr.push(key.split('active')[1]);
                    }
                }
            };
        },
        //btn-删除
        removeLine(){
            let deleteIdArr = [...this.deleteIdArr];
            if(deleteIdArr.length == 0) return ;
            this.deleteBtnDisable = true;
            for(let j=0; j<deleteIdArr.length; j++){
                this.$set(this.uploading, 'uploading'+deleteIdArr[j], true);
            };
            
            /***异步上传中***/
            setTimeout(()=>{
                if(Math.random() > 0.4){
                    success.call(this);
                }else{
                    faild.call(this);
                }
            }, 1000);

            // 成功的回调
            function success(){
                let d = [...this.data];
                let deleteIdArr = [...this.deleteIdArr];
                deleteIdArr.map(item=>{
                    item = item + "";
                    return item;
                });
                for(let j=0; j<deleteIdArr.length; j++){
                    this.$set(this.uploading, 'uploading'+deleteIdArr[j], false);
                };
                for(let [key, value] of Object.entries(this.checkboxControl)){
                    deleteIdArr.forEach(id=>{
                        if(key.split('active')[1] == id){
                            delete this.checkboxControl[key];
                        }
                    })
                };
                let _d = [];
                _d = d.filter(item=>{
                    if(deleteIdArr.includes(item.id + "")){
                        return false;
                    }
                    return true;
                });
                this.deleteIdArr = [];
                this.data = combineData(cleanData(_d));
                // this.data = _d;
                this.deleteBtnDisable = false;
            };
            // 失败的回调
            function faild(){
                for(let j=0; j<this.deleteIdArr.length; j++){
                    this.$set(this.uploading, 'uploading'+this.deleteIdArr[j], false);
                };
                this.deleteBtnDisable = false;
                console.log('删除失败！')
            };
        },

    },
};

function cleanData(data) {
    let data2 = [...data];
    let levelLength = 0;
    let clean = []
    if(data2.length == 0) return ;
    data2.forEach(item=>{
        if(item.level > levelLength){
            levelLength = item.level;
        }
    });
    data2 = data2.map(item=>{
        item.children = [];
        item.show = true;
        return item;
    });
    for(let i=0; i<=levelLength; i++){
        clean[i] = [];
        data2.forEach(item=>{
            if(item.level == i){
                clean[i].push(item)
            }
        });
    };
    for(let i=clean.length-1; i>=0; i--){
        for(let j=0; j<clean[i].length; j++){
            addToChildren(clean[i][j], i)
        }
    };
    return clean[0];
    function addToChildren(obj, index){
        if(index != 0){
            let _i = index - 1;
            for(let i=0; i<clean[_i].length; i++){
                if(clean[_i][i].id == obj.parentid){
                    clean[_i][i].icon = true;//初始化全部展开
                    clean[_i][i].children.push(obj);
                }
            }
        }
    };
}

function combineData(cleanData){
    let clean = [...cleanData];
    let res = [];
    for(let i=0; i<clean.length; i++){
        res.push(clean[i])
        a(clean[i])
    };
    return res;
    function a(whenClean){
        if(whenClean.children.length != 0){
            for(let j=0; j<whenClean.children.length; j++){
                res.push(whenClean.children[j]);
                a(whenClean.children[j]);
            }
        }
    }
}

</script>

<style lang="scss" scoped>
.tree{
    // overflow-x: hidden;
    .table-header{
        display: flex;
        width: 100%;
        li{
            padding: 0px 10px;
            background-color: #eef1f6;
            border: 1px solid #dfe6ec;
            color: #5c5c5c;
            font-weight: bold;
            font-size: 15px;
            text-indent: 12px;
            min-width: 90px;
            height: 36px;
            line-height: 36px;
        }
    }
    .table-body-li-wrap{
        position: relative;
        .uploading{
            position: absolute;
            height: 32px;
            width: 100%;
            background: rgba(255, 255, 255, .77);
            text-align: left;
            z-index: 1000;
            line-height: 32px;
            font-size: 20px;
            padding-left: 50px;
        }
        &:hover{
            background-color: #dfe6ec;
        }
    }
    .table-body{
        display: block;
        .table-body-li{
            padding: 0px 10px;
            background-color: rgba(255,255,255,.3);
            border: 1px solid #dfe6ec;
            color: #444;
            font-size: 14px;
            min-width: 90px;
            height: 32px;
            line-height: 32px;
            text-align: left;
            &.modify{
                background-color: rgba(206,142,64,.3);
            }
            .fa{
                cursor: pointer;
                &:hover{
                    color: #359899;
                }
            }
            input{
                max-width: 100%;
            }
            span.checkbox-span{
                width: 18px;
                height: 18px;
                margin-top: 7px;
                cursor: pointer; 
                border-radius: 50%;
                display: inline-block;
                position: relative;
                &.active{
                    .fa{
                        color: #49bea9;
                    }
                }
                .fa{
                    position: absolute;
                    font-size: 18px;
                    &:hover{
                        color: #49bea9;
                    }
                    left: 1px;
                    top: 1px;
                }
            }
        }
    }
    button{
        border: none;
        height: 18px;
        cursor: pointer;
        font-size: 12px;
    }
}
</style>

