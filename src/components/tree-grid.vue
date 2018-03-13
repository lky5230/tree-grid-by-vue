<template>
<div ref="tree" class="tree-wrap">
  <div class="tree">
    <div class="long-line" v-show="longShow" :style="{left: longLeft+'px'}"></div>
    <!--表头-->
    <ul class="table-header">
        <li
            class="nowrap"
            v-for="(col, colIndex) in column"
            :key="colIndex"
            :style="[{
                width: col.width? col.width+'px': minWidth+'px',
                flex: col.width? 'none': col.delete == true || col.operate == true? 'none': 1,
                overflow: col.operate == true? 'visible': '',
                padding: '0px 10px',
            }]">
            <template v-if="col.operate == true">
                <i style="transform: translateX(-12px)" class="fa fa-cogs" aria-hidden="true"></i>
            </template>
            <span v-show="col.delete == true">
                <button :style="{display: showDeleteBtn? '': 'none'}" id="tree_grid___id" title="删除所选项" class="btn-delete" :disabled="deleteBtnDisable" @click="removeLine">
                    <i v-if="deleteBtnDisable" class="fa fa-spin fa-spinner" aria-hidden="true"></i>
                    <i v-if="!deleteBtnDisable" class="fa fa-trash" aria-hidden="true"></i>
                </button>
            </span>
            <span v-if="!col.operate && !col.delete">
                <span v-if="col.isTree && showDeleteBtn">
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
                        width: col.width? col.width+'px': minWidth+'px',
                        flex: col.width? 'none': col.delete == true || col.operate == true? 'none': 1,
                        padding: '0px 10px',
                        paddingLeft: col.isTree? 10 + rowItem.level * 30+'px': '10px',
                        overflow: col.operate == true? 'visible': '',
                    }]">
                    <!--树的伸缩按钮-->
                    <!-- 1、有子节点的加图标 -->
                    <template v-if="col.isTree">
                        <!-- 线 -->
                        <!-- v-line-h v-line-full v-line-top v-line-bottom -->
                        <i 
                            v-if="rowItem.level != 0"
                            v-for="(line, indexLine) in rowItem.level" 
                            :key="indexLine"
                            :style="{left: (line-1)*30 + 16 +'px'}" 
                            class="v-line-full">
                        </i>
                        <i 
                            v-if="rowItem._icon_ == true" 
                            :style="{left: rowItem.level*30 + 16 +'px'}" 
                            class="v-line-bottom">
                        </i>
                        <i 
                            v-if="rowItem.level != 0"
                            :style="{left: (rowItem.level-1)*30 + 16 +'px'}" 
                            class="v-line-h">
                        </i>
                    </template>

                    <template v-if="col.isTree && rowItem.children.length != 0">
                        <span
                            @click="expand(rowItem, true)"
                            v-show="rowItem._icon_ == false">
                            <i class="fa fa-plus-square" aria-hidden="true"></i>
                        </span>
                        <span
                            @click="expand(rowItem, false)"
                            v-show="rowItem._icon_ == true">
                            <i class="fa fa-minus-square" aria-hidden="true"></i>
                        </span>
                    </template>
                    <!-- 2、需要动态增加子节点的加图标 -->
                    <template v-if="col.isTree && rowItem.children.length == 0 && rowItem.isleaf == 0">
                        <span @click.stop="updateLine(rowItem.id, rowItem.level)">
                            <i class="fa fa-plus-square" aria-hidden="true"></i>
                        </span>
                    </template>
                    <template v-if="col.isTree && rowItem.children.length == 0 && rowItem.isleaf == 1">
                        <i class="fa fa-file-text-o" aria-hidden="true"></i>
                    </template>

                    <!--不可编辑-->
                    <span v-if="!col.edit && !col.operate && !col.delete && col.prop != 'id' && !col.select && !col.funcList"> {{rowItem[col.prop]}} </span>
                    <span v-if="!col.edit && !col.operate && !col.delete && col.prop == 'id' && !col.select && !col.funcList"> {{rowItem[col.prop] | timestamp__}} </span>

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

                    <!--下拉-->
                    <template v-if="col.select">
                        <span
                            style=" text-align: left;"
                            :title="rowItem[col.prop]"
                            v-show="!operateStatus['status'+rowItem.id]"
                            class="no-wrap">
                            {{rowItem[col.prop]}}
                        </span>
                        <select
                            v-show="operateStatus['status'+rowItem.id]"
                            :style="{
                                width: col.isTree? 'auto': '100%'
                            }"
                            :value="lineValue['lineValue'+col.prop+rowItem.id]"
                            @change="inputEdit(rowItem.id, col.prop, $event)"
                            >
                            <option
                                v-for="(opt, index) in col.optionList"
                                :key="opt+index"
                                :value="opt">
                                {{opt}}
                            </option>
                        </select>
                    </template>

                    <!--功能选择组-->
                    <template v-if="col.funcList">
                      <label v-for="(func, index) in rowItem[col.prop]" :key="index" style="margin-right: 4px;">
                        <input
                          style="height: 12px;"
                          :ref="'check_row'+rowIndex+'col'+colIndex+'index'+index"
                          @change="(e)=>{funcChange(rowItem.id,rowIndex,colIndex,index,func.name)}"
                          :checked="func.checked"
                          type="checkbox" />
                        {{func.name}}
                      </label>
                    </template>

                    <!--操作按钮-->
                    <template v-if="col.operate">
                        <span class="hover-btn"
                            @mouseenter="mouseHover(rowItem.id, 1)"
                            @mouseleave="mouseHover(rowItem.id, 2)"
                            @click.stop="mouseClick(rowItem.id)"
                            >
                            <span class="hover-btn-span">
                                <i class="fa fa-pencil-square" aria-hidden="true"></i>
                            </span>
                            <transition name="as">
                                <span v-show="hoverBtn['hoverBtn'+rowItem.id]" class="col-btn-group">
                                    <span v-show="!operateStatus['status'+rowItem.id]">
                                        <button class="btn-edit" @click.stop="operateBtn(rowItem.id)">
                                            <i class="fa fa-pencil" aria-hidden="true"></i>&nbsp;编辑
                                        </button>
                                        <template v-if="rowItem.children.length == 0 && rowItem.isleaf == 0">
                                            <span style="font-size: 12px; color: #216a9c;">请先点击左侧 + 展开</span>
                                        </template>
                                        <template v-else>
                                            <button v-if="!onlyLineEdit" class="btn-add" @click.stop="addLine(rowItem.id, 1)">
                                                <i class="fa fa-plus" aria-hidden="true"></i>&nbsp;增加同级
                                            </button>
                                            <button v-if="!onlyLineEdit" class="btn-add" @click.stop="addLine(rowItem.id, 2)">
                                                <i class="fa fa-plus-circle" aria-hidden="true"></i>&nbsp;增加下级
                                            </button>
                                        </template>


                                        <button class="btn-upload" v-if="rowItem.modify" @click.stop="upload(rowItem.id)">
                                            <i class="fa fa-cloud-upload" aria-hidden="true"></i>&nbsp;上传
                                        </button>
                                    </span>
                                    <span v-show="!!operateStatus['status'+rowItem.id]">
                                        <button class="btn-confirm" @click.stop="saveBtn(rowItem.id)">
                                            <i class="fa fa-check" aria-hidden="true"></i>&nbsp;确认
                                        </button>
                                        <button class="btn-cancel" @click.stop="cancel(rowItem.id)">
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
                            <i class="fa fa-floppy-o" aria-hidden="true"></i>
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
        needFixWidth: { default: 0 },
        leafUrl: {},
        funcListAlias: {}, //用于http请求时，功能性复选框-名称对应
        showDeleteBtn: { default: false },
        onlyAddOne: { default: false },
        onlyLineEdit: { default: false },
        //刷新loading
        treeLoading: { default: false },
    },
    data() {
        return {
            column: [],
            data:[],
            //拖拽控制线位置
            longLeft: 0,
            //拖拽控制线显示
            longShow: false,
            minWidth: 50,
            device: null,
            // 每行数据对象的格式
            data_format: null,
            // 移入移出-控制
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
        //给列添加宽度
        this.column = [...this.columns];
        this.column.forEach(col=>{
          if(!col.width || isNaN(col.width)){
            col.width = this.minWidth;
          }
        });
        if(/Android|webOS|iPhone|iPod|BlackBerry/i.test(window.navigator.userAgent)) { this.device = 'phone' } else { this.device = 'pc' };
    },
    mounted(){
        this.fixedLastWidth();
    },
    watch: {
        needUpdate(){
            this.init();
    },
        needFixWidth(){
            this.fixedLastWidth();
        },
    },
    methods: {
        //修复最后一列宽度
        fixedLastWidth(){
            let tree_width = this.column.reduce((a, b, index) => index == 1? a.width + b.width: a + b.width);
            let tree = document.querySelector('.tree-wrap > .tree');
            if(tree_width < tree.offsetWidth){
                this.column[this.column.length - 1].width += tree.offsetWidth - tree_width;
            };
        },
        //刷新表格
        refreshTable(){
            this.refreshBtnDisable = true;
            this.deleteBtnDisable = true;
            this.$emit('refreshTable');
        },
        //初始化
        init(){
            if(this.rowdata.length == 0) return ;
            this.refreshBtnDisable = false;
            this.deleteBtnDisable = false;
            // 整理data
            this.data = combineData(cleanData([...this.rowdata]));
            this.$emit('currentDate', [...this.data])
            // 得到其中的数据对象格式
            if(this.data.length == 0) return ;
            let data_format = JSON.parse(JSON.stringify(this.data[0]));
            //默认值（数组的funcList保持原样！）
            initValue(data_format, '');
            function initValue(obj, init = ''){
                for(let [key, value] of Object.entries(obj)){
                    if(value == null) {
                        obj[key] = init;
                    }else{
                        if(typeof value != 'object'){
                            obj[key] = init;
                        }else{
                            if(value instanceof Array){
                                if(key != 'funcList'){
                                    for(let i=0; i<value.length; i++){
                                        initValue(value, init);
                                    }
                                }else{
                                    let vvv = [...value];
                                    for(let i=0; i<vvv.length; i++){
                                        vvv[i] = Object.assign(vvv[i]);
                                        vvv[i].checked = false;
                                    };
                                    obj[key] = vvv;
                                }
                            }else{
                                initValue(value, init);
                            }
                        }
                    }
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
                    if(col[j].edit == true || col[j].select == true){
                        // 初始化单元格input
                        this.$set(this.lineValue, 'lineValue'+col[j].prop+row[i].id, row[i][col[j].prop]);
                    }
                }
            }
        },
        //操作列-移入移出
        mouseHover(rowid, index){
            if(index == 1){
                this.$set(this.hoverBtn, 'hoverBtn'+rowid, true);
            }else if(index == 2){
                this.$set(this.hoverBtn, 'hoverBtn'+rowid, false);
            }
        },
        //兼容移动端-操作列
        mouseClick(rowid){
            let _this = this;
            if(this.device == 'phone'){
                this.mouseHover(rowid, 1);
                window.document.removeEventListener('click', docClick);
                window.document.addEventListener('click', docClick);
                function docClick(){
                    for(let key in _this.hoverBtn){
                        _this.$set(_this.hoverBtn, key, false);
                    };
                }
            };
        },
        //表头列-伸缩宽度
        willDragStart(col, e){
            e.preventDefault();
            if(col.operate) return ;
            this.startClientX = e.clientX;
            this.colItem = { ...col };
            let _this = this;
            let _width = col.width;
            let willWidth = col.width;
            //控制线
            let _column0 = [..._this.column];
            let _column0index = 0;
            for(let i=0; i<_column0.length; i++){
              _column0index += (_column0[i].width || 50)*1;
              if(_column0[i].prop == col.prop){
                _this.longLeft = _column0index;
                _this.longShow = true;
                break;
              }
            };
            let will_i = 0;
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
                        will_i = i;
                        willWidth = _width*1 + dis*1;
                        _this.longLeft = _column0index + dis;
                        break;
                    }
                };
            };
            function dragUp(){
              if(willWidth < _this.minWidth){
                _this.longShow = false;
                return ;
              }
              _this.column[will_i].width = willWidth;
              _this.longShow = false;
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
                    d[i]._icon_ = bool;
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
                                if(d[k]._icon_ == false){
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
            this.$emit('currentDate', [...this.data])
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
            this.$emit('currentDate', [...this.data])
            this.$set(this.lineValue, 'lineValue'+colProp+rowItemId, e.target.value);
        },
        //btn-编辑
        operateBtn(rowItemId){
            let d = [...this.data];
            let col = [...this.column];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    d[i].openEdit = true;
                    break;
                }
            };
            this.data = d;
            this.$emit('currentDate', [...this.data])
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
                    d[i].openEdit = false;
                    for(let j=0; j<col.length; j++){
                        if(col[j].edit == true || col[j].select == true){
                            d[i][col[j].prop] = this.lineValue['lineValue'+col[j].prop+rowItemId];
                        }
                    };
                    break;
                }
            };
            this.data = d;
            this.$emit('currentDate', [...this.data])
            //关闭编辑状态
            this.$set(this.operateStatus, 'status'+rowItemId, false)
        },
        //btn-撤销 
        cancel(rowItemId){
            let d = [...this.data];
            let col = [...this.column];
            for(let i=0; i<d.length; i++){
                if(d[i].id == rowItemId){
                    d[i].openEdit = false;
                    for(let j=0; j<col.length; j++){
                        if(col[j].edit == true || col[j].select == true){
                            this.lineValue['lineValue'+col[j].prop+rowItemId] = d[i][col[j].prop]
                        }
                    };
                    break;
                }
            };
            this.data = [...d];
            this.$emit('currentDate', [...this.data])
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
            };
            this.mouseHover(rowItemId, 2);
            function success(tip){
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
                this.$emit('currentDate', [...this.data])
                this.$set(this.uploading, 'uploading'+rowItemId, false);
                this.creatTipDom(tip || '上传成功！', 'success');
            }
            function faild(tip){
                this.$set(this.uploading, 'uploading'+rowItemId, false);
                this.creatTipDom(tip || '上传失败，请重试！', 'error');
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
                //增加项
                let addItem = {
                    ...JSON.parse(JSON.stringify(this.data_format)),
                    ...{
                        id: newId,
                        level: rowItem.level,
                        parentid: rowItem.parentid,
                        modify: true,
                        show: true,
                        isleaf: 1,
                        openEdit: true,
                        children: [],//长度为0，就没有+/-
                    }
                };
                //增加项的父层
                let _parentItem = d.filter(item=> item.id == addItem.parentid)[0];
                if(_parentItem != undefined){
                    //增加项的复选框状态
                    this.$set(this.checkboxControl, 'active'+addItem.id, this.checkboxControl['active'+_parentItem.id]);
                    //增加项的复选框状态为选中，则加入待删集合
                    if(this.checkboxControl['active'+addItem.id]){
                        this.deleteIdArr.push(addItem.id);
                    };
                };
                nowData.splice(lastLine, 0, addItem);
                this.$set(this.operateStatus, 'status'+newId, true); //打开编辑状态
                this.data = nowData;
                this.$emit('currentDate', [...this.data])
            }else if(indexed == 2){
                // 2、增加下级
                let d = [...this.data];
                let rowItem = null;
                let rowIndex = null;
                for(let i=0; i<d.length; i++){
                    if(d[i].id == rowItemId){
                        d[i].children = [true];
                        rowItem = d[i];
                        rowItem._icon_ = true;
                        rowIndex = i;
                    };
                };
                if(this.onlyAddOne){
                    if(rowItem.id.indexOf('_timestamp') != -1){
						if(this.$alert != undefined){
							this.$alert('请先保存节点')
						}else{
							window.alert('请先保存节点')
						};
                        return ;
                    };
                };
                let newId = Date.now()+'__timestamp__;';
                let [lastLine, nowData] = this.expand(rowItem, true);
                let addItem = {
                    ...JSON.parse(JSON.stringify(this.data_format)),
                    ...{
                        id: newId,
                        level: Number.parseInt(rowItem.level) + 1,
                        parentid: rowItem.id,

                        modify: true,
                        show: true,
                        isleaf: 1,
                        openEdit: true,

                        children: [],//长度为0，就没有+/-
                    }
                };
                //增加项的父层
                let _parentItem = d.filter(item=> item.id == addItem.parentid)[0];
                //增加项的复选框状态
                this.$set(this.checkboxControl, 'active'+addItem.id, this.checkboxControl['active'+_parentItem.id]);
                //增加项的复选框状态为选中，则加入待删集合
                if(this.checkboxControl['active'+addItem.id]){
                    this.deleteIdArr.push(addItem.id);
                };
                nowData.splice(Number.parseInt(rowIndex) + 1, 0, addItem);
                this.$set(this.operateStatus, 'status'+newId, true); //打开编辑状态
                this.data = nowData;
                this.$emit('currentDate', [...this.data])
            }
        },
        //btn-图标-更新 "isleaf = 0"
        updateLine(rowItemId, level){
            if(this.leafUrl == undefined) {
              console.warn('缺少leafUrl字段！')
              return ;
            };
            if(this.$http == undefined) {
              console.warn('请安装vue-resource！')
              return ;
            };
            this.mouseHover(rowItemId, 2);
            this.$set(this.uploading, 'uploading'+rowItemId, true);
            this.$http.get(this.leafUrl + rowItemId).then(res => {
                let d = [];
                if(res.data instanceof Array){
                    d = res.data;
                }else if(res.data.data instanceof Array){
                    d = res.data.data;
                }else if(res.data.data.result instanceof Array){
                    d = res.data.data.result;
                }else{
                    console.warn('请检查响应数据是否在response.data或response.data.data或response.data.data.result里面！');
                };
                if(d.length == 0){
                    //没有子节点数据，修改isleaf和children
                    let data = [...this.data];
                    let rowIndex = null;
                    for(let i=0; i<data.length; i++){
                        if(data[i].id == rowItemId){
                            data[i].children = [];
                            data[i].isleaf = 1;
                            break ;
                        };
                    };
                    this.data = data;
                    this.$emit('currentDate', [...this.data])
                    this.$set(this.uploading, 'uploading'+rowItemId, false);
                    return ;
                };
                this.deleteIdArr = [];
                let old_bool = this.checkboxControl['active'+rowItemId];
                // this.funcListAlias['funcList'] -> permission
                // this.funcListAlias['id'] -> permission_id
                // this.funcListAlias['name'] -> permission_name
                // this.funcListAlias['checked'] -> select
                for(let i=0; i<d.length; i++){
                    d[i] = {...JSON.parse(JSON.stringify(this.data_format)), ...d[i]};
                    if(this.funcListAlias){
                        d[i].funcList = d[i][this.funcListAlias['funcList']];
                        for(let k=0; k<d[i].funcList.length; k++){
                            d[i].funcList[k].id = d[i].funcList[k][this.funcListAlias['id']]
                            d[i].funcList[k].name = d[i].funcList[k][this.funcListAlias['name']]
                            d[i].funcList[k].checked = !!d[i].funcList[k][this.funcListAlias['checked']]
                        }
                    };
                    d[i].show = true;
                    d[i].level = level*1 + 1;
                    d[i].parentid = rowItemId;
                    this.$set(this.checkboxControl, 'active'+d[i].id, old_bool)
                };
                let data = [...this.data];
                let rowIndex = null;
                for(let i=0; i<data.length; i++){
                    if(data[i].id == rowItemId){
                        rowIndex = i;
                        data[i].children = [true];
                        data[i].isleaf = 1;
                        data[i]._icon_ = true;
                    };
                };
                data.splice(rowIndex*1+1, 0, ...d);
                this.data = data;
                this.$set(this.uploading, 'uploading'+rowItemId, false);
                //修复isleaf通过http得到的行，编辑时值为空的bug
                this.initValue();
                this.$emit('currentDate', [...this.data])
            }).catch(err=>{
                console.error(err);
                this.$set(this.uploading, 'uploading'+rowItemId, false);
                this.creatTipDom(`请求id：${rowItemId}失败！`,'error');
            });
        },
        //复选框切换
        checkbox_change(rowItemId){
            if(this.deleteBtnDisable) return;
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
            function success(tip){
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
                            delete _d[i]._icon_;
                            _d[i].children = [];
                        }
                    }else if(_d[i+1] == undefined){
                        delete _d[i]._icon_;
                        _d[i].children = [];
                    }
                };
                this.deleteIdArr = [];
                this.deleteBtnDisable = false;
                this.data = _d;
                this.$emit('currentDate', [...this.data])
                this.creatTipDom(tip || '删除成功！', 'success');
            };
            // 失败的回调
            function faild(tip){
                for(let j=0; j<this.deleteIdArr.length; j++){
                    this.$set(this.uploading, 'uploading'+this.deleteIdArr[j], false);
                };
                this.deleteBtnDisable = false;
                this.creatTipDom(tip || '删除失败，请重试！', 'error');
            };
        },
        //创建-提示框dom
        creatTipDom(tip, status){
            let tipDom = null;
            let tipBg = null;
            if(status == 'success'){
                tipDom = `<i class="fa fa-check-circle" aria-hidden="true"></i>&nbsp; ${tip}`;
                tipBg = '#40b965';
            }else if(status == 'error'){
                tipDom = `<i class="fa fa-times-circle" aria-hidden="true"></i>&nbsp;${tip}`;
                tipBg = '#da8b29';
            };
            let dom = window.document.createElement('div');
            dom.className = "__tip-dom__-wrap";
            dom.innerHTML = `<div class="__tip-dom__" style="text-align: center;height: 32px;padding-left: 12px;padding-right: 12px;line-height: 32px;background: ${tipBg};color: #fff;position: fixed;z-index: 10010;left: 50%;transform: translateX(-50%);top: -42px; transition: all .4s;"> ${tipDom} </div>`;
            this.$refs.tree.appendChild(dom);
            setTimeout(()=>{
                window.document.querySelector('.__tip-dom__').style.top = '14px';
                setTimeout(()=>{
                    window.document.querySelector('.__tip-dom__').style.top = '-42px';
                    setTimeout(()=>{
                        let remove = window.document.querySelector('.__tip-dom__-wrap');
                        this.$refs.tree.removeChild(remove);
                    }, 500);
                }, 2000);
            }, 0);
        },
        //功能组选择
        funcChange(rowid, rowIndex, colIndex, index, funcName){
          let inp = this.$refs['check_row'+rowIndex+'col'+colIndex+'index'+index]
          let checked = inp[0].checked;
          let d = [...this.data];
          for(let i=0; i<d.length; i++){
            if(d[i].id == rowid){
              if(funcName == d[i].funcList[index].name){
                d[i].modify = true;
                d[i].funcList[index].checked = checked;
              };
              break
            };
          };
          this.data = d;
          this.$emit('currentDate', [...this.data])
        },

    },
};

function cleanData(data) {
    let data2 = [...data];
    let levelLength = 0;
    let clean = []
    if(data2.length == 0) return [];
    data2 = data2.map(item=>{
        delete item.level;
        return item;
    });
    function convert(orgin)
    {
        var result = arguments[1] ? arguments[1] : [];
        var level = arguments[2] ? arguments[2] : 0;
        var parentid = arguments[3] ? arguments[3] : 0;
        for(var x in orgin)
        {
            if(orgin[x]["parentid"]==parentid&&orgin[x]["skip_012ea834e2aa011ef16f7d846889c026"]!=1)
            {
                orgin[x]["level"]=level;
                result.push(JSON.parse(JSON.stringify(orgin[x]))); //对象深拷贝
                orgin[x]["skip_012ea834e2aa011ef16f7d846889c026"]=1;//标记该节点不用再遍历
                if(orgin[x]["isleaf"]!=1)
                {
                   convert(orgin,result,level+1,orgin[x]["id"]);
                }
            }
        }
        return result;
    }
    data2 = convert(data2);

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
                    clean[_i][i]._icon_ = true;//初始化全部展开
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
*{ box-sizing: border-box !important; }
.nowrap{
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
.tree-wrap{
    position: relative;
    font-family: Helvetica Neue,Helvetica,PingFang SC,Hiragino Sans GB,Microsoft YaHei,SimSun,sans-serif;
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
    position: relative;
    .long-line{
      position: absolute;
      width: 1px;
      height: 100%;
      background: #aaa;
      z-index: 200;
      top: 0px;
      left: 0px;
    }
    .no-data{
        padding-top: 16px;
        padding-bottom: 16px;
        border: 1px solid #d8dcdf;
        color: rgb(94, 115, 130);
        font-size: 14px;
        text-align: center;
    }
    .table-header{
        display: flex;
        width: 100%;
        li{
            padding: 0px 10px;
            background-color: #eef1f6;
            border: 1px solid #dfe6ec;
            border-right: none;
            color: #1f2f3d;
            font-weight: 400;
            font-size: 16px;
            text-indent: 12px;
            height: 36px;
            line-height: 36px;
            position: relative;
            &:nth-last-child(1){
                border-right: 1px solid #dfe6ec;
            }
        }
        .w-resize{
            cursor: col-resize;
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
            background-color: #ebf3f0;
            .table-body-li{
                color: #965757;
            }
        }
    }
    .table-body{
        display: block;
        .table-body-li{
            background-color: rgba(255,255,255,.3);
            border: 1px solid #dfe6ec;
            border-top: none;
            border-right: none;
            color: #454545;
            font-size: 14px;
            height: 32px;
            line-height: 32px;
            text-align: left;
            position: relative;
            &:nth-last-child(1){
                border-right: 1px solid #dfe6ec;
            }
            &:nth-of-type(1){
                position: relative;
            }
            &.modify{
                background: #f7f4ea;
                color: #E3431C;
                .hover-btn-span{
                    color: #E3431C !important;
                }
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
                    color: #E3431C;
                    border: 1px solid #E3431C;
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
                top: calc(50%);
                right: 6px;
                transform: translateY(-50%);
                .modify-pin-title{
                    display: block;
                    height: 16px;
                    font-size: 10px;
                    line-height: 16px;
                    text-align: center;
                    background: #e76e50;
                    width: 54px;
                    color: #fff;
                }
                .fa{
                    display: block;
                    text-align: center;
                    font-size: 12px;
                    color: #E3431C;
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
        &.btn-update{
            background-color: #21c2a5;
            color: #eef1f6;
            &:hover{
                background-color: #2dcaae;
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
.v-line-full{
    position: absolute;
    top: 0px;
    left: 6px;
    height: 32px;
    width: 1px;
    background: #666;
}
.v-line-bottom{
    position: absolute;
    bottom: 0px;
    left: 6px;
    height: 10px;
    width: 1px;
    background: #666;
}
.v-line-h{
    position: absolute;
    top: 16px;
    left: 6px;
    height: 1px;
    width: 24px;
    background: #666;
}
</style>

