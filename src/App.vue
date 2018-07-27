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
        <template v-if="selectedMenuIndex === '1' ">
          <template v-if="pageData">

            <el-tabs v-model="activeTab" @tab-click="handleClick">
              <el-tab-pane v-for="tab in pageData" :key="tab.id" :label="tab.name" :name="tab.id">
                <template v-for="(table , i  ) in tab.son">
                  <div v-if="table.id !== '9'" class="table">
                    <div class="table-header"> {{ i }} -- {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-cell" v-for="(cell ,i) in  table.son" @click="addSelectedCells(cell)"
                           :style="{'background':_showCellBgc(cell)}">
                        <span>{{++i}}</span>
                        <span>{{cell.name}}</span>
                        <span><input type="number" v-model="cell.val"></span>
                      </div>
                    </div>
                  </div>
                  <div v-if="table.id === '9'" class="table table_9">
                    <div class="table-header"> {{ i }} -- {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-row" v-for="(row , i) in  table.son">
                        <span class="table-cell">{{row.name}}</span>
                        <span class="table-cell" v-for="(item , i) in row.sub">{{item}} <input type="checkbox"
                                                                                               v-model="item.val"></span>
                        <span class="table-cell">赔率：{{row.rate}}</span>
                        <span class="table-cell"> <input type="number">  </span>
                      </div>
                    </div>
                  </div>


                </template>

              </el-tab-pane>
            </el-tabs>


          </template>
          <template v-else>
            后台数据获取中...
          </template>
        </template>
        <template v-if="selectedMenuIndex === '2'">
          menu2 对应的view
        </template>
        <template v-if="selectedMenuIndex === '3'">
          menu3 对应的view
        </template>
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
        api: {
          host: "http://jon.linxizhilian.cn",
          pageData: "/pagedata"
        },
        activeTab: 'second',
        pageData: null,
        selectedMenuIndex: "1",
        selectedCells: []
      };
    },
    mounted() {
      var that = this;
      Axios.get(`${that.api.host}${that.api.pageData}`).then(
        (res) => {
          console.log(res.data)
          res.data.forEach(function (tab, i) {
            tab.son.forEach(function (table, i) {
              table.son.forEach(function (item) {
                item.val = ''
              })

            })
          });
          that.pageData = res.data
          that.activeTab = that.pageData[0].id;
        }
      )
    },
    computed: {},
    methods: {
      handleClick(tab, event) {
        console.log(tab, event);
      },
      _showCellBgc(cell) {
        return this.selectedCells.some(function (v, i) {
          return v === cell
        })
      },
      menuSelect(key, keyPath) {
        console.log(" menuSelect:", key, keyPath)
        this.selectedMenuIndex = key;
      },
      addSelectedCells(cell) {
        const that = this;
        const isExist = that.selectedCells.some(function (v) {
          return v === cell
        })

        if (!isExist) {
          that.selectedCells.push(cell)
        } else {
          let index = that.selectedCells.forEach(function (v, i) {
            if (v === cell) {
              return i
            }
          })
          that.selectedCells.splice(index, 1)
        }

        console.log(that.selectedCells)

      }
    }
  }
</script>

<style lang="less">

  #app {
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
        padding-right: 30px;
        .table {
          display: flex;
          flex-direction: column;
          margin-bottom: 20px;
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
              width: 33.333%;
              /*border: 1px solid seagreen;*/
              box-sizing: border-box;
              display: flex;
              flex-direction: row;
              justify-content: center;
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
                  width: 50px;
                }
                &:nth-child(1) {
                  background: #e7e7e7;
                  width: 20%;
                  font-weight: bolder;
                }

                &:nth-child(2) {
                  width: 20%;
                  color: red;
                  font-weight: bolder;
                }
                &:nth-child(3) {
                  text-align: left;
                  width: 60%;
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
              justify-content: space-around;
              > .table-cell {
                /*width: 12.5%;*/
                display: flex;
                flex-direction: row;
                align-content: center;
                > input {
                  display: inline-flex;
                }
              }
            }
          }
        }

      }

    }
  }
</style>
