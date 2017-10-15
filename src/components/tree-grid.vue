<template>
<div class="tree-wrap">
  <div class="tree">
    <!--表头-->
    <ul class="table-header">
        <li 
            class="nowrap"
            v-for="(col, colIndex) in column" 
            :key="colIndex"
            :style="[{
                width: col.width? col.width+'px': col.delete == true || col.operate == true? '50px': '',
                flex: col.width? 'none': col.delete == true || col.operate == true? 'none': 1,
                padding: col.operate == true || col.delete == true? '0px': '0px 10px',
                'min-width': col.operate == true || col.delete == true? '40px': '100px',
            }]">
            <template v-if="col.operate == true">
                <i style="transform: translateX(-12px)" class="fa fa-cogs" aria-hidden="true"></i>
            </template>
            <span v-else-if="col.delete == true">
                <button title="删除所选项" class="btn-delete" :disabled="deleteBtnDisable" @click="removeLine">
                    <i v-if="deleteBtnDisable" class="fa fa-spin fa-spinner" aria-hidden="true"></i>
                    <i v-if="!deleteBtnDisable" class="fa fa-trash" aria-hidden="true"></i>
                </button>
            </span>
            <span v-else>
                <span v-if="col.isTree">
                    <button 
                        title="刷新"
                        :disabled="refreshBtnDisable" 
                        @click="refreshTable"
                        class="btn-refresh">
                        <i 
                            :class="{'fa-spin':refreshBtnDisable? true: false}" 
                            class="fa fa-refresh" 
                            aria-hidden="true"></i>
                    </button>
                    &nbsp;
                </span>
                {{col.name}}
            </span>
            <span @mousedown="willDragStart(col, $event)" class="w-resize"></span>
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
                    :style="[{
                        width: col.width? col.width+'px': col.delete == true || col.operate == true? '50px': '',
                        flex: col.width? 'none': col.delete == true || col.operate == true? 'none': 1,
                        padding: col.operate == true? '0px': '0px 10px',
                        'padding-left': col.isTree? 24 * rowItem.level+14+'px': col.operate == true? '0px': '14px',
                        overflow: col.operate == true? 'visible': '',
                        'min-width': col.operate == true || col.delete == true? '40px': '100px',
                    }]">
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
                    <span v-if="!col.edit && !col.isTree && !col.operate && !col.delete && col.prop != 'id'"> {{rowItem[col.prop]}} </span>
                    <span v-if="!col.edit && !col.isTree && !col.operate && !col.delete && col.prop == 'id'"> {{rowItem[col.prop] | timestamp__}} </span>
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
                            :style="{
                                width: col.isTree? 'auto': '100%'
                            }"
                            @input="inputEdit(rowItem.id, col.prop, $event)"
                            :value="lineValue['lineValue'+col.prop+rowItem.id]" />
                    </template>

                    <!--操作按钮-->
                    <template v-if="col.operate">
                        <span class="hover-btn" @mouseover="mouseHover(rowItem.id, 1)"  @mouseout="mouseHover(rowItem.id, 2)" >
                            <span class="hover-btn-span">
                                &nbsp;<i class="fa fa-pencil-square" aria-hidden="true"></i>
                            </span>
                            <transition name="as">
                                <span v-show="hoverBtn['hoverBtn'+rowItem.id]" class="col-btn-group">
                                    <span v-show="!operateStatus['status'+rowItem.id]">
                                        <button class="btn-edit" @click="operateBtn(rowItem.id)">
                                            <i class="fa fa-pencil" aria-hidden="true"></i>&nbsp;编辑
                                        </button>
                                        <button class="btn-add" @click="addLine(rowItem.id, 1)">
                                            <i class="fa fa-plus" aria-hidden="true"></i>&nbsp;增加同级
                                        </button>
                                        <button class="btn-add" @click="addLine(rowItem.id, 2)">
                                            <i class="fa fa-plus-circle" aria-hidden="true"></i>&nbsp;增加下级
                                        </button>
                                        <button class="btn-upload" v-if="rowItem.modify" @click="upload(rowItem.id)">
                                            <i class="fa fa-cloud-upload" aria-hidden="true"></i>&nbsp;上传
                                        </button>
                                    </span>
                                    <span v-show="!!operateStatus['status'+rowItem.id]">
                                        <button class="btn-confirm" @click="saveBtn(rowItem.id)">
                                            <i class="fa fa-check" aria-hidden="true"></i>&nbsp;确认
                                        </button>
                                        <button class="btn-cancel" @click="cancel(rowItem.id)">
                                            <i class="fa fa-times" aria-hidden="true"></i>&nbsp;撤销
                                        </button>
                                    </span>
                                </span>
                            </transition>
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

                    <!--准备上传的提示-->
                    <template v-if="rowItem.modify && colIndex == 0">
                        <span class="modify-pin">
                            <span class="modify-pin-title">记得上传</span>
                            <i class="fa fa-exclamation" aria-hidden="true"></i>
                        </span>
                    </template>
                    
                </li>
            </div>
        </div>
    </ul>
    <!--空-->
    <div class="no-data" v-if="data.length == 0">
        <i class="fa fa-file-o" aria-hidden="true"></i>&nbsp;&nbsp;数据为空
    </div>
  </div>   
  <!--loading-->
  <div v-if="treeLoading && data.length != 0" class="tree-loading">
      <i class="fa fa-spinner fa-spin" aria-hidden="true"></i>
  </div>
  
</div> 
</template>

<script>

export default {
    name: 'treeGrid', 
    props: {
        columns:{ require: true, type: Array },
        rowdata:{ require: true, type: Array },
        needUpdate: { require: true },
        //刷新loading
        treeLoading: { default: false },
    },
    data() {
        return {
            column: [],
            data:[],
            // 每行数据对象的格式
            data_format: null,
            // 移入移出-控制
            operate:{},
            hoverBtn: {},
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
            //表头-刷新btn
            refreshBtnDisable: false,
        };
    },
    filters: {
        timestamp__: function(val){
            if((val + "").indexOf('__timestamp__') != -1){
                return ' - '
            };
            return val;
        }
    },
    created(){
        this.column = [...this.columns];
    },
    watch: {
        needUpdate(){
            this.init();
        }
    },
    methods: {
        //刷新表格
        refreshTable(){
            this.refreshBtnDisable = true;
            this.$emit('refreshTable');
        },
        //初始化
        init(){
            if(this.rowdata.length == 0) return ;
            this.refreshBtnDisable = false;
            // 整理data
            this.data = combineData(cleanData([...this.rowdata]));
            // 得到其中的数据对象格式
            let data_format = {...this.data[0]};
            for(let key in data_format){
                if(data_format.hasOwnProperty(key)){
                    data_format[key] = '';
                }
            };
            this.data_format = data_format;
            // 初始化行
            this.initValue();
        },
        //初始化行各单元格控制
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
        //操作列的移入移出
        mouseHover(rowid, index){
            if(index == 1){
                this.$set(this.hoverBtn, 'hoverBtn'+rowid, true);
            }else if(index == 2){
                this.$set(this.hoverBtn, 'hoverBtn'+rowid, false);
            }
        },
        //表头列-伸缩宽度
        willDragStart(col, e){
            if(col.prop == undefined) return ;
            if(col.width == undefined) return ;
            this.startClientX = e.clientX;
            this.colItem = { ...col };
            let _this = this;
            let _width = col.width;
            window.removeEventListener('mousemove', dragMove);
            window.removeEventListener('mouseup', dragUp);
            window.addEventListener('mousemove', dragMove);
            window.addEventListener('mouseup', dragUp);
            function dragMove(ev){
                _this.moveClientX = ev.clientX;
                let dis = _this.moveClientX - _this.startClientX;
                let _column = [..._this.column];
                for(let i=0; i<_column.length; i++){
                    if(_column[i].prop == col.prop){
                        _column[i].width = _width*1 + dis*1;
                        break;
                    }
                };
                _this.column = _column;
            };
            function dragUp(){
                window.removeEventListener('mousemove', dragMove);
                window.removeEventListener('mouseup', dragUp);
            };
        },
        //相关行的折叠与展开
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
                        if(d[k] == undefined) break;
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
        //btn-确认
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
                    this.$emit('uploadmodify', [d[i], success.bind(this), faild.bind(this)]);
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
                console.log('上传失败！')
                this.$set(this.uploading, 'uploading'+rowItemId, false);
            }
        },
        //btn-同下级的增加
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
                let newId = Date.now() + '__timestamp__';
                nowData.splice(lastLine, 0, {...this.data_format, ...{
                    id: newId,
                    level: rowItem.level,
                    parentid: rowItem.parentid,
                    
                    modify: true,
                    show: true,
                    
                    children: [],//长度为0，就没有+/-
                }});
                //打开编辑状态
                this.$set(this.operateStatus, 'status'+newId, true)
                this.data = nowData;
            }else if(indexed == 2){
                // 2、增加下级
                let d = [...this.data];
                let rowItem = null;
                let rowIndex = null;
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        d[i].children = [true];
                        rowItem = d[i];
                        rowItem.icon = true;
                        rowIndex = i;
                    };
                };
                let newId = Date.now()+'__timestamp__;';
                let [lastLine, nowData] = this.expand(rowItem, true);
                nowData.splice(Number.parseInt(rowIndex) + 1, 0, {...this.data_format, ...{
                    id: newId,
                    level: Number.parseInt(rowItem.level) + 1,
                    parentid: rowItem.id,
                    
                    modify: true,
                    show: true,
                    
                    children: [],//长度为0，就没有+/-
                }});
                this.$set(this.operateStatus, 'status'+newId, true);//打开编辑状态
                this.data = nowData;
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
                            if(d[k] == undefined) break;
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
                            if(d[k] == undefined) break;
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
            this.$emit('uploaddelete', [deleteIdArr, success.bind(this), faild.bind(this)]);
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
                let _d = d.filter(item=>{
                    for(let i=0; i<deleteIdArr.length; i++){
                        if(deleteIdArr[i]+"" == item.id+""){
                            return false;
                        }
                    };
                    return true; 
                });
                // 修正icon是否显示
                for(let i=0; i<_d.length; i++){
                    if(_d[i+1] != undefined){
                        if(_d[i+1].level <= _d[i].level){
                            delete _d[i].icon;
                            _d[i].children = [];
                        }
                    }else if(_d[i+1] == undefined){
                        delete _d[i].icon;
                        _d[i].children = [];
                    }
                };
                this.deleteIdArr = [];
                this.deleteBtnDisable = false;
                this.data = _d;
            };
            // 失败的回调
            function faild(){
                for(let j=0; j<this.deleteIdArr.length; j++){
                    this.$set(this.uploading, 'uploading'+this.deleteIdArr[j], false);
                };
                this.deleteBtnDisable = false;
                alert('删除失败！')
            };
        },

    },
};

function cleanData(data) {
    let data2 = [...data];
    let levelLength = 0;
    let clean = []
    if(data2.length == 0) return [];
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
        expandToArray(clean[i])
    };
    return res;
    function expandToArray(whenClean){
        if(whenClean.children.length != 0){
            for(let j=0; j<whenClean.children.length; j++){
                res.push(whenClean.children[j]);
                expandToArray(whenClean.children[j]);
            }
        }
    }
}

</script>

<style lang="scss" scoped>
*{
    box-sizing: border-box !important;
}
.nowrap{
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
.tree-wrap{
    position: relative;
    .tree-loading{
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0px;
        left: 0px;
        z-index: 10010;
        background-color: rgba(255, 255, 255, .77);
        .fa{
            font-size: 28px;
            display: block;
            text-align: center;
            margin-top: 52px; 
        }
    }
}
.tree{
    margin: 2px;
    .no-data{
        padding-top: 16px;
        padding-bottom: 16px;
        border: 1px solid #d8dcdf;
        color: #6e5f49;
    }
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
            position: relative;
        }
        .w-resize{
            cursor: w-resize;
            position: absolute;
            right: 0px;
            top: 0px;
            height: 32px;
            width: 8px;
            background: transparent;
            z-index: 101;
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
            background-color: rgba(255,255,255,.3);
            border: 1px solid #dfe6ec;
            color: #444;
            font-size: 14px;
            height: 32px;
            line-height: 32px;
            text-align: left;
            &:nth-of-type(1){
                position: relative;
                overflow: visible;
            }
            &.modify{
                background: #f7f4ea;
            }
            .fa{
                cursor: pointer;
                &:hover{
                    color: #359899;
                }
            }
            input{
                max-width: 100%;
                margin: 0px;
                height: 24px;
                border: 1px solid #aabdbe;
                text-indent: 8px;
                &:focus{
                    outline: none;
                    -moz-outline: none;
                    color: #378686; 
                    border: 1px solid #2eb4ba;
                }
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
            .hover-btn{
                position: relative;
                display: inline-block;
                width: 100%;
                .hover-btn-span{
                    display: inline-block;
                    width: 100%;
                    text-align: center;
                    font-size: 18px;
                    background: #eef1f6;
                    border: none;
                    color: #2472a9;
                    &:hover{
                        background: #d8e0ec;
                    }
                }
                .col-btn-group{
                    position: absolute;
                    left: calc(100% - 1px);
                    top: -2px;
                    z-index: 99;
                    background: #9adde8;
                    padding-left: 10px;
                    padding-right: 8px;
                    border: 1px solid #356fab;
                    &:after{
                        position: absolute;
                        content: ' ';
                        width: 12px;
                        height: 12px;
                        background: #9adde8;
                        z-index: 98;
                        left: -7px;
                        top: 10px;
                        transform: rotate(45deg);
                        border: 1px solid #356fab;
                        border-right: none;
                        border-top: none;
                    }
                }
            }
            .modify-pin{
                position: absolute;
                right: 0;
                bottom: calc(100% - 20px);
                perspective: 150px;
                animation: tip 5.2s infinite ease-in-out;
                transform-origin: center 22px;
                .modify-pin-title{
                    display: block;
                    height: 16px;
                    font-size: 10px;
                    line-height: 16px;
                    text-align: center;
                    background: #dfc888;
                    width: 54px;
                    color: #652e0c;
                }
                .fa{
                    display: block;
                    text-align: center;
                    font-size: 10px;    
                    color: #864816;
                }
            }
            
        }
    }
    button{
        border: none;
        height: 18px;
        cursor: pointer;
        font-size: 12px;
        padding-left: 6px;
        padding-right: 6px;
        &:hover,&:active,&:focus{
            outline: none;
            -moz-outline: none;
        }
        &.btn-delete{
            transform: translateX(-6px);
            background-color: #dbb602;
            color: #86340c;
            font-size: 15px;
            padding-left: 8px;
            padding-right: 8px;
            height: 20px;
            border: none;
            &:hover{
                background-color: #f3be91;
            }
            &[disabled]{
                background-color: #f3be91;
            }
        }
        &.btn-refresh{
            transform: translateX(-6px);
            background-color: #1dd545;
            color: #126868;
            font-size: 15px;
            padding-left: 8px;
            padding-right: 8px;
            height: 20px;
            border: none;
            &:hover{
                background-color: #7ff699;
            }
            &[disabled]{
                background-color: #a0eeb1;
            }
        }
        &.btn-edit{
            background-color: #2a75a9;
            color: #eef1f6;
            &:hover{
                background-color: #55a4db;
            }
        } 
        &.btn-add{
            background-color: #4f911d;
            color: #eef1f6;
            &:hover{
                background-color: #74ba3e;
            }
        } 
        &.btn-upload{
            background-color: #59ca1f;
            color: #eef1f6;
            &:hover{
                background-color: #6ddb35;
            }
        } 
        &.btn-confirm{
            background-color: #26a10f;
            color: #eef1f6;
            &:hover{
                background-color: #46d72b;
            }
        } 
        &.btn-cancel{
            background-color: #a99d0d;
            color: #eef1f6;
            &:hover{
                background-color: #cec120;
            }
        }
    }
    
}

@keyframes tip{
    22%{ transform: rotateZ(0deg); }
    24%{ transform: rotateZ(-4deg); }
    26%{ transform: rotateZ(4deg); }
    28%{ transform: rotateZ(0deg); }

    68%{ transform: translateY(0px); }
    71%{ transform: translateY(-3px); }
    74%{ transform: translateY(0px); }
    77%{ transform: translateY(-3px); }
    80%{ transform: translateY(0px); }
}
.as-enter-active, .as-leave-active {
  transition: all .2s
}
.as-enter, .as-leave-to {
  opacity: 0;
  transform: translateX(16px);
}
</style>

