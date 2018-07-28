<template>

  <el-container id="app">
    <el-header>Header</el-header>
    <el-container class="innerBox">
      <el-aside width="260px">
        <div>
          <!--  @open="handleOpen" @close="handleClose"  -->
          <el-menu :default-active="selectedMenuIndex" background-color="#eeeeee" text-color="#333"
                   active-text-color="#000" @select="menuSelect">
            <el-menu-item index="1">
              <span slot="title">最新投注</span>
              <i class="el-icon-menu"></i>
            </el-menu-item>
            <el-menu-item index="2">
              <i class="el-icon-menu"></i>
              <span slot="title">双面长龙</span>
            </el-menu-item>
            <el-menu-item index="3">
              <i class="el-icon-document"></i>
              <span slot="title">全部长龙</span>
            </el-menu-item>
            <el-menu-item index="4">
              <i class="el-icon-setting"></i>
              <span slot="title">开奖结果</span>
            </el-menu-item>
          </el-menu>
        </div>

      </el-aside>
      <el-main>
        <!--最新投注-->
        <template v-if="selectedMenuIndex === '1' ">
          <template v-if="pageData">
            <!--弹出层 确认订单 提交数据-->
            <el-dialog title="订单信息" :visible.sync="dialogVisible" width="40%">
              <div v-for="(item  , i ) in buyData">{{item.parent_name}} --- {{ item.val }}</div>
              <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="sendDataToBuy()">确 定</el-button>
              </span>
            </el-dialog>

            <!--操作栏-->
            <div class="caozuo">
              <div>快速输入：<input type="number" v-model="setAll" @keyup="setAllSelectedCells()"></div>
              <!--@keyup="setAllSelectedCells()"-->
              <el-button @click="filterUseFul()">确定</el-button>
              <el-button @click="reloadWindow()"> 重置 </el-button>
            </div>

            <el-tabs v-model="activeTab" @tab-click="tabsClick">
              <el-tab-pane v-for="tab in pageData" :key="tab.id" :label="tab.name" :name="tab.id">

                <template v-for="(table , i  ) in tab.son">

                  <!-- 表格： 三军 3 / 和值4 -->
                  <div v-if="table.id === '3' || table.id === '4'" class="table">
                    <div class="table-header"> {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-cell" v-for="(cell ,i) in  table.son" :class="{ bgYellow:cell.isSelected}">
                        <span>{{++i}}</span>
                        <span>{{cell.rate}}</span>
                        <span><input v-model="cell.val" type="number" @click.self="toggleSelected(cell)"></span>
                        <!--@change.self="checkInput(cell)"-->
                      </div>
                    </div>
                  </div>

                  <!-- 表格： 和值双面5 /  过关6  / 豹、顺、杂、紅黑码7  / 跨度8-->
                  <div v-if="table.id === '5' || table.id === '6' || table.id === '7' || table.id === '8'"
                       class="table">
                    <div class="table-header"> {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-cell" v-for="(cell ,i) in  table.son" :class="{ bgYellow:cell.isSelected}">
                        <span>{{cell.name}}</span>
                        <span>{{cell.rate}}</span>
                        <span><input type="number" v-model="cell.val" @click.self="toggleSelected(cell)"></span>
                      </div>
                    </div>
                  </div>


                  <!-- 表格： 红黑码 9-->
                  <div v-if="table.id === '9'" class="table table_9">
                    <div class="table-header"> {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-row" v-for="(row , i) in  table.son" :key="i"
                           :class="{ bgYellow:row.isSelected}">
                        <span class="table-cell" :style="{ color: row.name === '4码红'? 'red':'' }">{{row.name}}</span>
                        <div class="table-cell table-cell-small" v-for="v in row.sub">
                          <i>{{v}}</i><input type="checkbox" v-model="row.checked" :value="v">
                        </div>
                        <span class="table-cell" style="color: red">赔率：{{row.rate}}</span>
                        <span class="table-cell"> <span>金额：</span><input v-model="row.val" type="number"
                                                                         @click.self="toggleSelected(row)"> {{row.name}} </span>
                      </div>
                    </div>
                  </div>


                  <!-- 表格： 三同号单选59 /  二同号复选60 / 二同号单选61  /  三不同号62  / 二不同号63  -->
                  <div
                    v-if="table.id === '59' || table.id === '60'  || table.id === '61'  || table.id === '62' || table.id === '63'"
                    class="table">
                    <div class="table-header"> {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-cell" v-for="(cell ,i) in  table.son" :class="{ bgYellow:cell.isSelected}">
                        <span>{{cell.name}}</span>
                        <span>{{cell.rate}}</span>
                        <span><input type="number" v-model="cell.val" @click.self="toggleSelected(cell)"></span>
                      </div>
                    </div>
                  </div>

                </template>

              </el-tab-pane>
            </el-tabs>


          </template>
          <template v-else>
            {{tabsTip}}
          </template>
        </template>

        <!--双面长龙-->
        <template v-if="selectedMenuIndex === '2'">
          menu2 对应的view
        </template>

        <!--全部长龙-->
        <template v-if="selectedMenuIndex === '3'">
          menu3 对应的view
        </template>

        <!--开奖结果-->
        <template v-if="selectedMenuIndex === '4'">
          menu4 对应的view
        </template>

      </el-main>
    </el-container>


  </el-container>
</template>

<script>
  import Axios from 'axios';

  export default {
    name: 'App',
    data() {
      return {
        tabsTip: '后台数据获取中...',
        setAll: undefined,
        dialogVisible: false,
        api: {
          host: "http://jon.linxizhilian.cn",
          pageData: "/pagedata",
          buy: "/buyrecord",
          index: "/indexdata"
        },
        activeTab: 'second',
        pageData: null,
        buyData: [],
        indexData: undefined, //
        selectedMenuIndex: "1",
        selectedCells: [],

      };
    },
    mounted() {
//      console.log('mounted:');
      this.getPageData();
      this.getIndexData();
    },
//    beforeUpdate(){
//      console.log( 'beoforeUpdate')
//    },

    computed: {
      _calRate_simahei() {

      },
    },
    methods: {
      tabsClick(tab, event) {
        //tab 选项开切换时的调用
        console.log(tab)


      },
      setAllSelectedCells() {
        // 设置selectedCells.val = setAll
        const that = this;

        console.log('setAll:', that.setAll)

        if (!(that.selectedCells.length > 0)) {
          console.log('没有找到需要快速输入的元素')
          return
        }
//        if (!that.setAll) {
//          console.log('快速输入：值缺失')
//          return
//        }

        for (let i = 0; i < that.selectedCells.length; i++) {
          let cell = that.selectedCells[i]
          cell.val = that.setAll;
        }

        console.log('快速输入后 - selectedCells：', that.selectedCells)

//
//        console.log('that.setAll:', that.setAll)
//        that.clearSeletedCells()
//        console.log( '全部设置后的selectedCells:' , that.selectedCells )
      },
      checkInput(cell) {
        if (isNaN(cell.val)) {
          cell.val = ''
        }
        if (cell.val <= 0) {
          cell.val = ''
        }
      },
      reloadWindow() {
//        alert( 'reload')
//        this.pageData = '数据重新加载中...';
//        this.getPageData()
        window.location.reload()
      },
      clearSeletedCells() {
        //重置： 清空所有已经选择到的元素, 并设置其isSelected 为false

        const that = this;
        that.selectedCells.forEach(function (v) {
          v.isSelected = false;
          console.log(v.isSelected);
        })

        that.selectedCells.length = 0;
        that.setAll = undefined;


      },

      // 左侧menu菜单切换时的调用函数
      menuSelect(key, keyPath) {

        const that = this;

        that.selectedCells.forEach(function (cell) {
          that.toggleSelected(cell)
        })

        this.selectedMenuIndex = key;
      },


      toggleSelected(cell) {
//        let cell = cell;
        const that = this;
//        let isExist = that.selectedCells.some(function (v) {
//          return v === cell
//        })

        let isExist = undefined;

        for (let i = 0; i < that.selectedCells.length; i++) {
          if (that.selectedCells[i] === cell) {
            isExist = true
          }
        }


        if (!isExist) {
          that.$set(cell, 'isSelected', true)
//          cell.isSelected = true;
          that.selectedCells.push(cell)
        } else {
          that.$set(cell, 'isSelected', false)

          let index = undefined;
          for (let i = 0; i < that.selectedCells.length; i++) {
            if (that.selectedCells[i] === cell) {
              index = i
            }
          }

          if (!index && index !== 0) {
            console.log('toggleSelected - err: 存在但未匹配到')
          }

          that.selectedCells.splice(index, 1)
        }

//        that.$set(that , 'pageData' , data)

        that.setAllSelectedCells();

        console.log('selectedCells:', that.selectedCells)

      },


//      向后台获取pageData接口数据，并绑定至data.pageData
      getPageData() {
        const that = this;
        Axios.get(`${that.api.host}${that.api.pageData}`).then(
          (res) => {
//            console.log(res.data)
            res.data.forEach(function (tab, i) {
              tab.son.forEach(function (table, i) {

                if (table.id === '9') {
                  // 红黑码id：9    为数据添加checked 属性为空数组，方便记录checkBox的选择值;
                  table.son.forEach(function (item) {
                    item.val = '';
                    item.checked = [];
//                    item.limted = parseInt(item.name[0])
                  })
                } else {
                  table.son.forEach(function (item) {
                    item.val = ''
                  })
                }
              })
            });
            that.pageData = res.data;
            that.activeTab = that.pageData[0].id;
          }
        )
      },
      sendDataToBuy() {
        const that = this;
        const url = `${that.api.host}${that.api.buy}`;
        const data = that.buyData;
        console.log('sendData:', data)

        let config = {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
        };
        Axios.post(url, JSON.stringify(data), config).then(function (res) {
          console.log('success: to buyAPI ', res.data)
          console.log('success: to buyAPI ', res.data.status)

          if(res.data.status === 200){

          }
        }).catch(function (err) {
          console.log('err: to buyAPI ', err)
        })


        that.dialogVisible = false;
      },
      getIndexData() {
        const that = this;

        const url = `${that.api.host}${that.api.index}`
        Axios.get(url).then(function (res) {
//            console.log( 'success - getIndexData:' , res.data )

          that.indexData = res.data;
        }).catch(function (err) {
          console.log('err - getIndexData:', err)
        })
      },

      isCheckBoxOk() {
        const that = this;

        let flag = true;

        that.pageData.forEach(function (tab) {
          tab.son.forEach(function (table) {
            if (table.id === '9') {
              table.son.forEach(function (row) {
//                     console.log(row)
                if ((row.checked.length !== row.limit) && row.val) {
                  alert(`【${row.parent_name} - ${row.name}】：请选择${row.limit}个`)
                  flag = false;
                }
              })
            }
          })
        })

        return flag

      },

//      获取用户输入的有效数据绑定至data.buyData , 发送至后台
      filterUseFul() {
        const that = this;
        if (!that.isCheckBoxOk()) {
          return
        };

        that.pageData.forEach(function (tab, i) {
          tab.son.forEach(function (table, i) {
            table.son.forEach(function (cell, i) {
              let flag = that.buyData.some(function (item) {
                return item.id === cell.id
              })

              if (cell.val) {
                if (!flag) {
                  that.buyData.push(cell)
                }
              }
            })
          })
        })


        if( !that.buyData.length){
          return
        }
        that.dialogVisible = true;
//          that.sendDataToBuy()
      }

    },
  }
</script>

<style lang="less">

  #app {

    .bgYellow {
      background: yellow !important;
    }

    width: 100%;
    height: 100%;
    /*background: sandybrown;*/
    .el-header {
      /*background: seashell;*/
    }
    .innerBox {
      .el-aside {
        padding-top: 30px;
        background: #eeeeee;

      }
      .el-main {
        padding-left: 30px;
        /*padding-right: 30px;*/
        padding-right: 20%;

        .caozuo {
          display: flex;
          flex-direction: row;
          align-content: space-around;
          align-items: center;
        }
        .table {
          display: flex;
          flex-direction: column;
          margin-bottom: 5px;
          .table-header {
            text-align: center;
            font-weight: bolder;
            padding: 5px 0;
            /*background: #dadada;*/
            background-image: linear-gradient(to bottom, #dadada, rgba(232, 232, 232, 0.56), #dadada);
            border: 1px solid #a3a3a3;
          }
          .table-body {
            display: flex;
            flex-direction: row;
            flex-wrap: wrap;
            box-sizing: border-box;
            .table-cell {
              width: 20%;
              box-sizing: border-box;
              display: flex;
              flex-direction: row;
              justify-content: flex-start;
              align-content: center;
              align-self: center;
              text-align: center;
              height: 30px;
              border-bottom: 1px solid #b5b5b5;
              border-left: 1px solid #b5b5b5;
              border-right: 1px solid #b5b5b5;

              &:not(:first-child) {
                /*border-left:none ;*/
              }

              > span {
                /*border-left: 1px solid #b5b5b5 ;*/
                &:first-child {
                  border-left: none;
                }
                > input {
                  width: 60px;
                }
                &:nth-child(1) {
                  background: #e7e7e7;
                  width: 20%;
                  font-weight: bolder;
                }

                &:nth-child(2) {
                  /*width: 30%;*/
                  color: red;
                  font-weight: bolder;
                  padding-right: 10px;
                  min-width: 80px;
                }
                &:nth-child(3) {
                  text-align: left;
                  /*width: 50%;*/
                }
              }
            }
          }
        }
        .table_9 {
          > .table-body {
            display: flex;
            flex-direction: column;
            > .table-row {
              display: flex;
              flex-direction: row;
              align-items: center;
              /*justify-content: space-around;*/
              > .table-cell-small {
                width: 12% !important;
                display: flex;
                flex-direction: row;
                justify-content: flex-start;
                align-content: center;
                > input {
                  width: 18px;
                  height: 18px;
                }
                > i {
                  min-width: 20px;
                }

              }
              .table-cell {
                &:last-child {
                  /*background: salmon;*/
                  display: flex;
                  justify-content: flex-start;
                  align-items: center;
                  > span {
                    display: inline-flex;
                    min-width: 50px;
                    background: none;
                  }
                  > input {
                    width: 60px;
                  }
                }
                &:first-child {
                  font-weight: bolder;
                  padding-left: 6px;
                }
              }
            }
          }
        }

      }

    }
  }
</style>
