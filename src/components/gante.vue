<template>
  <div class="gante-warp">

    <div class="gante-left">
      <table class="left-table" cellspacing="0" cellpadding="0">
        <!--左侧头-->
        <tr class="thead">
          <th v-for="(thead, index) in ganttDate.leftName" :key="index">
            <div>{{thead}}</div>
          </th>
        </tr>
        <!--左侧体-->
        <tr class="tbody" v-for="(row, index) in ganttRowData" :key="index">
          <td v-for="(ganttLeft, index2) in ganttDate.leftName" :key="index2">
            <div :class="[row.isParent? 'parentLeft': 'norLeft']">{{row.left[ganttLeft]}}</div>
          </td>
        </tr>
      </table>
    </div>

    <div style="position: relative;">

      <span class="drag-cursor" @mousedown="willDragStart($event)"></span>

      <div class="gante-right" :style="{width: ganteRightWidth + 'px'}">
        <div>
          <table class="right-table-head" cellspacing="0" cellpadding="0">
            <!--右侧头-->
            <thead>
              <tr class="gante-head-year">
                <td v-for="(rH, index) in rightHead" :key="index" :colspan="rH.date.length">
                  <div>{{rH.yearmonth}}</div>
                </td>
              </tr>
              <tr class="gante-head-date">
                <template v-for="(rH, index) in rightHead">
                  <td v-for="(d, index2) in rH.date" :key="index2">
                    <div>{{d}}</div>
                  </td>
                </template>
              </tr>
            </thead>
            <!--右侧体-->
            <tbody>
              <tr
              class="tbody tbody-right"
              v-for="(ganttRow, index) in ganttRowData"
              :key="index"
              :ref=" 'ptr_' + ganttRow.id + '_' + (ganttRow.right.right_start || 'null') + '_' + (ganttRow.right.progress_date || 'null') + '_' + (ganttRow.right.right_stop || 'null')"
              >
                <!--父组-->
                <template v-if="ganttRow.isParent == true">
                  <td v-for="dd in disDate" :key="dd">
                    <div
                      :class="{
                        'progress-normal': dd >= ganttRow.right.progress_date && dd <= ganttRow.right.right_stop,
                        'progress': dd >= ganttRow.right.right_start && dd <= ganttRow.right.right_stop,
                      }">
                        <span>&nbsp;</span>
                      </div>
                  </td>
                </template>

                <!--子组-->
                <template v-else>
                  <td
                    v-for="(dd, ddIndex) in disDate"
                    :ref="ganttRow.id + '_' + dd" :key="dd">
                    <div v-if="isInStatusList(dd, ganttRow.right) == false">
                      &nbsp;
                    </div>
                    <div v-else :class="{
                        'A': isInStatusList(dd, ganttRow.right).status == 'A',
                        'B': isInStatusList(dd, ganttRow.right).status == 'B',
                        'C': isInStatusList(dd, ganttRow.right).status == 'C',
                        'D': isInStatusList(dd, ganttRow.right).status == 'D',
                      }">
                      <span
                        class="popover"
                        @mouseenter="$set(pop, ganttRow.id + '_' + dd, true)"
                        @mouseleave="$set(pop, ganttRow.id + '_' + dd, false)">

                        <i class="pop" v-if="pop[ganttRow.id + '_' + dd]">
                          {{dd}}
                          <br>
                          {{'进度：'+isInStatusList(dd, ganttRow.right).number}}
                        </i>

                        <i v-if="
                        (isInStatusList(dd, ganttRow.right).number+'').length
                            >  3"
                        >
                          {{(isInStatusList(dd, ganttRow.right).number+"").slice(0, 3)+'...'}}
                        </i>

                        <i v-else>
                          {{isInStatusList(dd, ganttRow.right).number}}
                        </i>

                      </span>
                    </div>
                  </td>
                </template>

              </tr>
            </tbody>
          </table>
        </div>
      </div>

    </div>

  </div>
</template>

<script>

const Snap = require(`imports-loader?this=>window,fix=>module.exports=0!snapsvg/dist/snap.svg.js`);
console.log(Snap)
export default {
  data() {
    return {
      //缓存数据
      disDate: [],
      //右侧被清理的表头列表
      rightHead: [],
      //pop
      pop: {},
      //右侧初始宽度
      ganteRightWidth: 0,

      //甘特表-全局
      ganttDate: {
        startDate: '2012-02-07', //起始日
        stopDate: '2012-03-12', //结束日
        loading: false, //甘特表的loading
        leftName: ['name', 'code'], //左侧显示哪些字段
      },

      //甘特表-所有的行数据
      ganttRowData: [
        {
          id: '1111', //行的id、编号
          status: 'normal', //行的状态
          showEdit: false, //显示编辑行的模态框
          isParent: true,
          left: { //左侧拥有的字段
            name: '撒大',
            code: '454',
            warn: 'sb2',
            sttr: 'sttr'
          },
          right: {
            //右侧数据的任务日期间隔[仅父组使用和拥有]
            right_start: '2012-02-12',
            right_stop: '2012-03-06',
            progress_date: '2012-02-19',
            //任务状态集合[仅子组使用和拥有]
            //statusList: ['A','B','A','A','B','C','C'],
          },
        },
        {
          id: '222',
          status: 'normal',
          showEdit: false,
          to: '444',
          left: {
            name: 'fff',
            code: '777',
            warn: 'zzcxs',
            sttr: 'sttr'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-14',
              status: 'B',
              number: 22,
            },{
              date: '2012-02-15',
              status: 'D',
              number: 33,
            },{
              date: '2012-02-16',
              status: 'D',
              number: 61,
            },{
              date: '2012-02-17',
              status: 'C',
              number: 23,
            },{
              date: '2012-02-18',
              status: 'A',
              number: 64,
            },{
              date: '2012-02-19',
              status: 'A',
              number: 21,
            },{
              date: '2012-02-20',
              status: 'B',
              number: 43,
            }],
          },
        },
        {
          id: '333',
          status: 'normal',
          showEdit: false,
          to: '444',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-16',
              status: 'A',
              number: 44,
            },{
              date: '2012-02-17',
              status: 'B',
              number: 13,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-19',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-20',
              status: 'B',
              number: 66,
            },{
              date: '2012-02-21',
              status: 'D',
              number: 29,
            }],
          },
        },
        {
          id: '444',
          status: 'normal',
          showEdit: false,
          left: {
            name: 'sda',
            code: 'jhg',
            warn: 'desdgtr',
            sttr: 'cxs'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-17',
              status: 'A',
              number: 56,
            },{
              date: '2012-02-18',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-19',
              status: 'B',
              number: 92,
            },{
              date: '2012-02-20',
              status: 'A',
              number: 12,
            },{
              date: '2012-02-21',
              status: 'C',
              number: 35,
            }],
          },
        },

        {
          id: '555555', //行的id、编号
          status: 'normal', //行的状态
          showEdit: false, //显示编辑行的模态框
          isParent: true,
          left: { //左侧拥有的字段
            name: '23撒大',
            code: 'bb454',
            warn: 'ddsb2',
            sttr: 'desttr'
          },
          right: {
            //右侧数据的任务日期间隔[仅父组使用和拥有]
            right_start: '2012-02-10',
            right_stop: '2012-03-01',
            progress_date: '2012-02-14',
            //任务状态集合[仅子组使用和拥有]
            //statusList: ['A','B','A','A','B','C','C'],
          },
        },
        {
          id: '98765',
          status: 'normal',
          showEdit: false,
          to: '999',
          left: {
            name: 'fff',
            code: '777',
            warn: 'zzcxs',
            sttr: 'sttr'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-12',
              status: 'C',
              number: 22,
            },{
              date: '2012-02-13',
              status: 'B',
              number: 231,
            },{
              date: '2012-02-14',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-15',
              status: 'C',
              number: 23,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 64,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 43,
            }],
          },
        },
        {
          id: '5342',
          status: 'normal',
          showEdit: false,
          to: '999',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-13',
              status: 'C',
              number: 44,
            },{
              date: '2012-02-14',
              status: 'C',
              number: 13,
            },{
              date: '2012-02-15',
              status: 'A',
              number: 91,
            },{
              date: '2012-02-16',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-18',
              status: 'D',
              number: 29,
            }],
          },
        },
        {
          id: '83454',
          status: 'normal',
          showEdit: false,
          to: '999',
          left: {
            name: '12543',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-12',
              status: 'D',
              number: 33,
            },{
              date: '2012-02-13',
              status: 'D',
              number: 23,
            },{
              date: '2012-02-14',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-15',
              status: 'C',
              number: 27,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-17',
              status: 'B',
              number: 3124675646,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 54,
            },{
              date: '2012-02-19',
              status: 'C',
              number: 231,
            },{
              date: '2012-02-20',
              status: 'D',
              number: 14,
            }],
          },
        },
        {
          id: '53413542',
          status: 'normal',
          showEdit: false,
          to: '999',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-11',
              status: 'B',
              number: 44,
            },{
              date: '2012-02-12',
              status: 'B',
              number: 13,
            },{
              date: '2012-02-13',
              status: 'A',
              number: 91,
            },{
              date: '2012-02-14',
              status: 'D',
              number: 27,
            },{
              date: '2012-02-15',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-16',
              status: 'B',
              number: 29,
            }],
          },
        },
        {
          id: '999',
          status: 'normal',
          showEdit: false,
          left: {
            name: 'sda',
            code: 'jhg',
            warn: 'desdgtr',
            sttr: 'cxs'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-15',
              status: 'B',
              number: 56,
            },{
              date: '2012-02-16',
              status: 'B',
              number: 17,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 92,
            }],
          },
        },

        {
          id: '432435432', //行的id、编号
          status: 'normal', //行的状态
          showEdit: false, //显示编辑行的模态框
          isParent: true,
          left: { //左侧拥有的字段
            name: '23撒大',
            code: 'bb454',
            warn: 'ddsb2',
            sttr: 'desttr'
          },
          right: {
            //右侧数据的任务日期间隔[仅父组使用和拥有]
            right_start: '2012-02-10',
            right_stop: '2012-03-01',
            progress_date: '2012-02-14',
            //任务状态集合[仅子组使用和拥有]
            //statusList: ['A','B','A','A','B','C','C'],
          },
        },
        {
          id: '8542543',
          status: 'normal',
          showEdit: false,
          to: '345',
          left: {
            name: 'fff',
            code: '777',
            warn: 'zzcxs',
            sttr: 'sttr'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-12',
              status: 'C',
              number: 22,
            },{
              date: '2012-02-13',
              status: 'B',
              number: 231,
            },{
              date: '2012-02-14',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-15',
              status: 'C',
              number: 23,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 64,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 43,
            }],
          },
        },
        {
          id: 'fsar435',
          status: 'normal',
          showEdit: false,
          to: '345',
          left: {
            name: 'fff',
            code: '777',
            warn: 'zzcxs',
            sttr: 'sttr'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-12',
              status: 'C',
              number: 22,
            },{
              date: '2012-02-13',
              status: 'B',
              number: 231,
            },{
              date: '2012-02-14',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-15',
              status: 'C',
              number: 23,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 64,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 11,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 43,
            }],
          },
        },
        {
          id: '43kkgf',
          status: 'normal',
          showEdit: false,
          to: '345',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-13',
              status: 'C',
              number: 44,
            },{
              date: '2012-02-14',
              status: 'C',
              number: 13,
            },{
              date: '2012-02-15',
              status: 'A',
              number: 91,
            },{
              date: '2012-02-16',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-18',
              status: 'D',
              number: 29,
            }],
          },
        },
        {
          id: '6534g',
          status: 'normal',
          showEdit: false,
          to: '345',
          left: {
            name: '12543',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-12',
              status: 'D',
              number: 33,
            },{
              date: '2012-02-13',
              status: 'D',
              number: 23,
            },{
              date: '2012-02-14',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-15',
              status: 'C',
              number: 27,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-17',
              status: 'B',
              number: 3124675646,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 54,
            },{
              date: '2012-02-19',
              status: 'C',
              number: 231,
            },{
              date: '2012-02-20',
              status: 'D',
              number: 14,
            }],
          },
        },
        {
          id: 'y4536t42',
          status: 'normal',
          showEdit: false,
          to: '345',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-11',
              status: 'B',
              number: 44,
            },{
              date: '2012-02-12',
              status: 'B',
              number: 13,
            },{
              date: '2012-02-13',
              status: 'A',
              number: 91,
            },{
              date: '2012-02-14',
              status: 'D',
              number: 27,
            },{
              date: '2012-02-15',
              status: 'A',
              number: 66,
            },{
              date: '2012-02-16',
              status: 'B',
              number: 29,
            }],
          },
        },
        {
          id: '345',
          status: 'normal',
          showEdit: false,
          left: {
            name: 'sda',
            code: 'jhg',
            warn: 'desdgtr',
            sttr: 'cxs'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-15',
              status: 'B',
              number: 56,
            },{
              date: '2012-02-16',
              status: 'B',
              number: 17,
            },{
              date: '2012-02-17',
              status: 'A',
              number: 92,
            }],
          },
        },

        {
          id: '543422dfs', //行的id、编号
          status: 'normal', //行的状态
          showEdit: false, //显示编辑行的模态框
          isParent: true,
          left: { //左侧拥有的字段
            name: '撒大',
            code: '454',
            warn: 'sb2',
            sttr: 'sttr'
          },
          right: {
            //右侧数据的任务日期间隔[仅父组使用和拥有]
            right_start: '2012-02-12',
            right_stop: '2012-03-06',
            progress_date: '2012-02-17',
            //任务状态集合[仅子组使用和拥有]
            //statusList: ['A','B','A','A','B','C','C'],
          },
        },
        {
          id: 'dfsxgfsav43',
          status: 'normal',
          showEdit: false,
          to: '111',
          left: {
            name: 'fff',
            code: '777',
            warn: 'zzcxs',
            sttr: 'sttr'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-14',
              status: 'B',
              number: 22,
            },{
              date: '2012-02-15',
              status: 'D',
              number: 33,
            },{
              date: '2012-02-16',
              status: 'D',
              number: 61,
            },{
              date: '2012-02-17',
              status: 'C',
              number: 23,
            },{
              date: '2012-02-18',
              status: 'A',
              number: 64,
            },{
              date: '2012-02-19',
              status: 'A',
              number: 21,
            },{
              date: '2012-02-20',
              status: 'B',
              number: 43,
            }],
          },
        },
        {
          id: '65fddgsdf',
          status: 'normal',
          showEdit: false,
          to: '111',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-16',
              status: 'A',
              number: 44,
            },{
              date: '2012-02-17',
              status: 'B',
              number: 13,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-19',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-20',
              status: 'B',
              number: 66,
            },{
              date: '2012-02-21',
              status: 'D',
              number: 29,
            }],
          },
        },
        {
          id: 'dsa35476ujhgfd',
          status: 'normal',
          showEdit: false,
          to: '111',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-16',
              status: 'A',
              number: 44,
            },{
              date: '2012-02-17',
              status: 'B',
              number: 13,
            },{
              date: '2012-02-18',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-19',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-20',
              status: 'B',
              number: 66,
            },{
              date: '2012-02-21',
              status: 'D',
              number: 29,
            }],
          },
        },
        {
          id: '765342gfxbvdsabgdfcdsx',
          status: 'normal',
          showEdit: false,
          to: '111',
          left: {
            name: '54353',
            code: 'jghhf',
            warn: 'sb2',
            sttr: 'fdsd'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-13',
              status: 'D',
              number: 44,
            },{
              date: '2012-02-14',
              status: 'C',
              number: 13,
            },{
              date: '2012-02-15',
              status: 'B',
              number: 91,
            },{
              date: '2012-02-16',
              status: 'A',
              number: 321,
            }],
          },
        },
        {
          id: '111',
          status: 'normal',
          showEdit: false,
          left: {
            name: 'sda',
            code: 'jhg',
            warn: 'desdgtr',
            sttr: 'cxs'
          },
          right: {
            //任务状态集合[仅子组使用和拥有]
            statusList: [{
              date: '2012-02-17',
              status: 'A',
              number: 56,
            },{
              date: '2012-02-18',
              status: 'C',
              number: 17,
            },{
              date: '2012-02-19',
              status: 'B',
              number: 92,
            },{
              date: '2012-02-20',
              status: 'A',
              number: 12,
            },{
              date: '2012-02-21',
              status: 'C',
              number: 35,
            }],
          },
        },
      ],
    };
  },

  props: {

  },

  created() {
    this.init()
  },

  mounted() {
    this.ganteRightWidth = window.document.body.offsetWidth * (3 / 4);
    this.createLine();

  },

  methods: {
    //左右侧宽度拉伸
    willDragStart(e) {
      let orgX = e.clientX;
      let _this = this;
      let orgWidth = _this.ganteRightWidth;
      window.document.addEventListener('mousemove', move);
      window.document.addEventListener('mouseup', up);
      function move(e2) {
        _this.disX = e2.clientX - orgX;
        _this.ganteRightWidth = orgWidth - _this.disX;
      };
      function up() {
        window.document.removeEventListener('mousemove', move);
        window.document.removeEventListener('mouseup', up);
      };
    },
    //初始化
    init() {
      this.cleanRightDate();
    },

    //右侧头日期数据处理
    cleanRightDate() {
      let getDate = this.getDisDate(
        this.ganttDate.startDate,
        this.ganttDate.stopDate
      );
      this.disDate = getDate;
      this.rightHead = clean(getDate);
      function clean(dateArr) {
        let res = [];
        let yearMonth = Array.from(new Set([...dateArr].map(d => d.slice(0, 7)))).sort();
        for (let i = 0; i < yearMonth.length; i++) {
          res[i] = {
            yearmonth: yearMonth[i],
            date: [],
          };
          for (let j = 0; j < dateArr.length; j++) {
            if (dateArr[j].indexOf(yearMonth[i]) > -1) {
              res[i].date.push(dateArr[j].slice(8));
            }
          }
          res[i].date.sort();
        };
        return res;
      };
    },
    //获取两个日期间的所有日期
    getDisDate(start, end) {
      function getDate(datestr) {
        let temp = datestr.split("-");
        let date = new Date(temp[0], temp[1] - 1, temp[2]);
        return date;
      }
      let date_all = [];
      let i = 0;
      let startTime = getDate(start);
      let endTime = getDate(end);
      while ((endTime.getTime() - startTime.getTime()) >= 0) {
        let year = startTime.getFullYear();

        let month = (startTime.getMonth() + 1).toString().length == 1?
        "0" + (startTime.getMonth() + 1).toString():
        (startTime.getMonth() + 1).toString();

        let day = startTime.getDate().toString().length == 1?
        "0" + startTime.getDate():
        startTime.getDate();

        date_all[i] = year + "-" + month + "-" + day;
        startTime.setDate(startTime.getDate() + 1);
        i += 1;
      }
      return date_all;
    },
    //判断日期是否在该StatusList对象数组中,并返回对应项
    isInStatusList(date, right){
      if(right.statusList == undefined) return false;
      for(let i=0; i<right.statusList.length; i++){
        if(right.statusList[i].date == date){
          return right.statusList[i];
        };
      };
      return false;
    },
    //生成坐标指向、间距
    createCoord(){
      let d = [...this.ganttRowData];
      let clean = [];
      for(let i=0; i<d.length; i++){
        if(d[i].isParent) continue;
        //垂直间隔
        let myIndex = i;
        let toIndex = 0;
        for(let j=0; j<d.length; j++){
          if(d[j].id == d[i].to){
            toIndex = j;
            break ;
          }
        };
        clean.push({
          id: d[i].id,
          to: d[i].to? d[i].to: 0,
          vCount: toIndex == 0? null: toIndex - myIndex,
          start: d[i].right.statusList[0].date,
          stop: d[i].right.statusList[d[i].right.statusList.length - 1].date
        });
      };
      return clean;
    },
    //根据坐标指向绘制子组曲线
    createLine(){
      let warp = window.document.querySelector('.right-table-head');
      let svg = Snap(warp.offsetWidth, warp.offsetHeight);
      let cellWidth = 34;
      let cellHeight = 24;
      svg.node.style.position = 'absolute';
      svg.node.style.left = '0px';
      svg.node.style.top = '0px';
      svg.node.style['z-index'] = '-1';
      warp.appendChild(svg.node);

      let d = this.createCoord();
      let p = [];
      let ids = [];
      for(let i=0; i<d.length; i++){
        if(d[i].to != 0){
          p.push(d[i].stop);
          ids.push(d[i].id);
        }else{
          let maxDate = p[0];
          let minDate = p[0];
          for(let j=0; j<p.length; j++){
            if(maxDate < p[j]){
              maxDate = p[j];
            };
            if(minDate > p[i]){
              minDate = p[i];
            }
          };
          //画子组
          drew.call(this, maxDate, minDate, ids, d[i]);
          ids = [];
          p = [];
        };
      };
      //画父组
      let progressColor = ['#37c020', '#ccc9bb'];

      let pd = [...this.ganttRowData].filter(item => item.isParent == true);
      for(let i=0; i<pd.length; i++){

        let ptr = this.$refs['ptr_'+pd[i].id+'_'+pd[i].right.right_start+'_'+pd[i].right.progress_date+'_'+pd[i].right.right_stop][0];

        //绿色
        let horCount = getDateDiff(
          pd[i].right.right_start,
          pd[i].right.progress_date
        );
        let startTd = ptr.querySelector('.progress').parentNode;
        let left = startTd.offsetLeft;
        let top = startTd.offsetTop;
        let greenPath = svg.paper.path(`M${left},${top*1 + 12}L${left*1+cellWidth*horCount*1+5},${top*1 + 12}`)
        .attr({
          stroke: progressColor[0],
          strokeWidth: cellHeight - 8,
          fill: 'none'
        });
        let length = Snap.path.getTotalLength(greenPath);
        greenPath.attr({
          'stroke-dashoffset': length,
          'stroke-dasharray': length
        });
          Snap.animate(length, 0, function(val) {
            greenPath.attr({
            'stroke-dashoffset': val
            });
          }, 1000, mina.linear());

        //灰色
        let horCountEnd = getDateDiff(
          pd[i].right.progress_date,
          pd[i].right.right_stop
        );
        let endTd = ptr.querySelector('.progress-normal').parentNode;
        let left2 = endTd.offsetLeft;
        let top2 = endTd.offsetTop;
        let greyPath = svg.paper.path(`M${left2},${top2*1 + 12}L${left2*1+cellWidth*horCountEnd*1+5},${top2*1 + 12}`)
        .attr({
          stroke: progressColor[1],
          strokeWidth: cellHeight - 8,
          fill: 'none'
        });
        let length2 = Snap.path.getTotalLength(greyPath);
        greyPath.attr({
          'stroke-dashoffset': length2,
          'stroke-dasharray': length2
        });
        setTimeout(()=>{
          Snap.animate(length2, 0, function(val) {
            greyPath.attr({
            'stroke-dashoffset': val
            });
          }, 1000, mina.easeinout());
        }, 1000 );

      };

      function drew(maxDate, minDate, ids, lastObj){
        let d2 = [];
        for(let i=0; i<d.length; i++){
          for(let j=0; j<ids.length; j++){
            if(d[i].id == ids[j]){
              d2.push(d[i]);
            }
          }
        };
        let canDrew = false;
        let drewIndex = -1;
        let colorArray = ['#a026e9', '#ff2e2e', '#31d016', '#258dd9', '#f5a40b'];
        let colorIndex = rd(0, colorArray.length - 1);
        for(let i=0; i<d2.length; i++){
          if(d2[i].to != 0){

            let horizonCount = getDateDiff(d2[i].stop, maxDate);
            if(canDrew == false && d2[i].stop == minDate){
              canDrew = true;
              drewIndex = i;
            };
            let td = this.$refs[d2[i].id + '_' + d2[i].stop][0];
            let l = td.offsetLeft+cellWidth + 4;
            let t = td.offsetTop+cellHeight / 2 + 2;
            let color = colorArray[colorIndex];

            //圆点
            let cir1 = svg.paper.circle(l, t*1 + 1, 1).attr({
              fill: color,
              cx: l
            });
            setTimeout(()=>{
              cir1.animate({ r: 3.5 }, 1400, mina.bounce); },
              Math.min(500 * (i*1 + 1), 1800)
            );

            //线条（注意减掉边框）
            if(i == drewIndex){
              let path = svg.paper.path(`
                  M${l}, ${t*1 + 1}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount}, ${t*1 + 1}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount}, ${t * 1 + 1 + cellHeight * 1 / 2 + cellHeight * (d2[i].vCount - 1)}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount - cellWidth * (getDateDiff(lastObj.start, maxDate) * 1 + 2)}, ${t * 1 + 1 + cellHeight * 1 / 2 + cellHeight * (d2[i].vCount - 1)}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount - cellWidth * (getDateDiff(lastObj.start, maxDate) * 1 + 2)},${t * 1 + 1 + cellHeight * 1 / 2 + cellHeight * (d2[i].vCount - 1) + cellHeight / 2}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount - cellWidth * (getDateDiff(lastObj.start, maxDate) * 1 + 2) + cellWidth / 2 - 4},
                  ${t * 1 + 1 + cellHeight * 1 / 2 + cellHeight * (d2[i].vCount - 1) + cellHeight / 2}`)
                .attr({
                  stroke: color,
                  strokeWidth: 2,
                  fill: 'none'
                });
                let length = Snap.path.getTotalLength(path);
                path.attr({
                  'stroke-dashoffset': length,
                  'stroke-dasharray': length
                });
                setTimeout(()=>{
                  Snap.animate(length, 0, function(val) {
                    path.attr({
                    'stroke-dashoffset': val
                    });
                  }, 1500, mina.easeout());
                }, Math.min(600*rd(0, 8), 1800) );
            }else{
              let path = svg.paper.path(
                `M${l}, ${t*1 + 1}L${l * 1 + cellWidth / 2 + cellWidth * horizonCount}, ${t*1 + 1}`
              ).attr({
                stroke: color,
                strokeWidth: 2,
                fill: 'none'
              });
              let length = Snap.path.getTotalLength(path);
              path.attr({
                'stroke-dashoffset': length,
                'stroke-dasharray': length
              });
              setTimeout(()=>{
                Snap.animate(length, 0, function(val) {
                  path.attr({
                  'stroke-dashoffset': val
                  });
                }, 1500, mina.easeout());
              }, Math.min( 250*rd(0, 2), 500) );
            };

            if(i == drewIndex){
              svg.paper.circle(
                l * 1 + cellWidth / 2 + cellWidth * horizonCount - cellWidth * (getDateDiff(lastObj.start, maxDate) * 1 + 2) + cellWidth / 2 - 9,
                t * 1 + cellHeight * 1 / 2 + cellHeight * (d2[i].vCount - 1) + cellHeight / 2 + 1,
                2
              ).attr({
                fill: color
              });
            };

          };
        };
        function rd(n,m){
            let c = m - n + 1;
            return Math.floor(Math.random() * c + n);
        };
      };

      function getDateDiff(startDate,endDate){
        let startTime = new Date(Date.parse(startDate.replace(/-/g, "/"))).getTime();
        let endTime = new Date(Date.parse(endDate.replace(/-/g, "/"))).getTime();
        let dates = Math.abs((startTime - endTime)) / (1000 * 60 * 60 * 24);
        return dates;
      };
    },

  },

  components: {},

}

</script>

<style lang='scss' scoped>

$headHeight: 50px;
$trHeight: 24px;
$cellWidth: 34px;

*{ box-sizing: border-box };

.gante-warp {
  width: 100%;
  position: relative;
  display: flex;
  align-items: stretch;

  .drag-cursor {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 6px;
    cursor: w-resize;
    background: #75def5;
    z-index: 666;
  }
  .gante-left {
    flex: auto;
    border: 1px solid rgba(222, 222, 222, .75);
    overflow-x: auto;
    .left-table {
      min-width: 100%;
      .thead {
        height: $headHeight;
        line-height: $headHeight;
        border-bottom: 1px solid rgba(222, 222, 222, .75);
        border-top: 1px solid rgba(222, 222, 222, .75);
        div {
          padding-left: 6px;
          padding-right: 6px;
          min-width: 120px;
        }
      }
    }
  }
  .gante-right {
    border: 1px solid rgba(222, 222, 222, .75);
    flex: none;
    position: relative;
    padding-left: 5px;
    overflow-x: auto;
    height: 100%;
    .right-table-head {
      position: relative;
      .gante-head-year {
        height: $headHeight/2;
        line-height: $headHeight/2;
        td {
          div {
            min-width: 66px;
            text-align: center;
            border: 1px solid rgba(222, 222, 222, .75);
            border-right: none;
            color: #2299c8;
          }
        }
      }
      .gante-head-date {
        height: $headHeight/2;
        line-height: $headHeight/2;
        td {
          div {
            width: $cellWidth;
            text-align: center;
            border: 1px solid rgba(222, 222, 222, .75);
            border-right: none;
            border-top: none;
          }
        }
      }
    }
  }
  .tbody{
    height: $trHeight;
    line-height: $trHeight;
    border-bottom: 1px solid rgba(222, 222, 222, .75);
    color: #4d4d4d;
    .parentLeft{
      text-indent: 16px;
      color: #b4620a;
    }
    .norLeft{
      text-indent: 32px;
    }
    td{
      border-left: 1px solid rgba(222, 222, 222, .75);
    }
  }
  .tbody-right{
    text-align: center;
    .popover{
      position: relative;
      .pop{
        position: absolute;
        left: -33px;
        bottom: 24px;
        width: calc(100% + 66px);
        padding: 3px 6px;
        background-color: rgba(82, 75, 94, .92);
        &:after{
          position: absolute;
          content: ' ';
          width: 10px;
          height: 10px;
          background-color: rgba(82, 75, 94, .92);
          transform: rotate(45deg);
          bottom: -5px;
          left: calc(50% - 5px);
        }
      }
    }
    td{
      height: $trHeight;
      line-height: 18px;
    }
    div{
      height: $trHeight;
      width: $cellWidth;
      color: #fff;
      font-size: 12px;
      span{
        width: calc(100% - 0px);
        display: inline-block;
        border-radius: 0px;
        height: $trHeight - 8px;
        position: relative;
        top: 4px;
      }
    }
    .progress{
      span{

      }
    }
    .progress-normal{
      span{

      }
    }

    .A{
      span{
        background: #ee890e;
      }
    }
    .B{
      span{
        background: #d4dd54;
      }
    }
    .C{
      span{
        background: #23f8ec;
      }
    }
    .D{
      span{
        background: #74353d;
      }
    }
  }
}
</style>
