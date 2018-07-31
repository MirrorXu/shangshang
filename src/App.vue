<template>

  <el-container id="app">
    <el-header>
      <div class="userInfo">
        <div class="userInfo-left">
            <a href="/">首页</a>
        </div>
        <div class="userInfo-right" >
          <div class="userInfo-name"><span>账号：</span><span v-if="userInfo">{{userInfo.username}}</span></div>
          <div class="userInfo-score"><span>积分：</span><span v-if="userInfo">{{userInfo.score}}</span></div>
          <div class="userInfo-logout"><a href="/logout">登出</a></div>
        </div>
      </div>
    </el-header>
    <el-container class="innerBox">
      <el-aside width="260px">
        <div>
          <!--  @open="handleOpen" @close="handleClose"  -->
          <el-menu :default-active="selectedMenuIndex" background-color="#eeeeee" text-color="#333"
                   active-text-color="#000" @select="menuSelect">
            <el-menu-item index="2">
              <i class="el-icon-menu"></i>
              <span slot="title">最新投注</span>
            </el-menu-item>
            <el-menu-item index="3">
              <i class="el-icon-setting"></i>
              <span slot="title">开奖结果</span>
            </el-menu-item>
          </el-menu>
        </div>
      </el-aside>


      <el-main>
        <!--最新投注-->
        <template v-if="selectedMenuIndex === '2' ">
          <template v-if="pageData">
            <!--弹出层 确认订单 提交数据-->
            <el-dialog title="订单信息" :visible.sync="dialogVisible" width="40%">
              <!--<div v-for="(item  , i ) in buyData">{{item.parent_name}} -&#45;&#45; {{ item.val }}</div>-->


              <template v-if="buyData.length > 0">
                <el-table :data="buyData" style="width: 100%">
                  <el-table-column label="种类" width="280">
                    <!--prop="parent_name"-->
                    <template slot-scope="scope">
                      <!--<i class="el-icon-time"></i>-->
                      <span style="margin-left: 10px" v-if="scope.row.parent_name === '红黑码'">{{ scope.row.parent_name
                        }} <span style="color:red">{{scope.row.checked}}</span></span>
                      <span style="margin-left: 10px"
                            v-else>{{ scope.row.parent_name.length > 15 ? scope.row.parent_name.slice(0, 12) + '...' : scope.row.parent_name
                        }}</span>
                    </template>
                  </el-table-column>
                  <el-table-column label="赔率" width="180" prop="rate">

                  </el-table-column>

                  <el-table-column label="金额" width="180" prop="val">

                  </el-table-column>
                  <el-table-column label="操作">
                    <template slot-scope="scope">
                      <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    </template>
                  </el-table-column>
                </el-table>
                <div class="dingdanMoney">
                  <div>账户余额：<span style="color: red">{{  userInfo.score }}</span></div>
                  <div>订单金额：<span style="color: red">{{  _calAllMoney }}</span></div>
                </div>
              </template>
              <template v-else>
                <div style="padding: 20px 0">
                  空空如也...
                </div>
              </template>


              <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="sendDataToBuy()" :disabled="buyData.length === 0">确 定</el-button>
              </span>
            </el-dialog>
            <!--操作栏-->

            <!--公共消息-->
            <div class="info">
              <div class="row_1" v-if="indexData ">
                <div class="left">
                  <div class="ret">
                    <span>开奖结果：</span>
                    <div class="bar">
                      <i @click="changelotteryIndex(1)"
                         class="el-icon-caret-left"></i><span> {{ indexData.lottery_data[lotteryIndex].periods
                      }}</span> <i @click="changelotteryIndex(-1)" class="el-icon-caret-right"> </i>
                    </div>
                    <div class="balls">
                      <span class="ball"
                            v-for=" v in indexData.lottery_data[lotteryIndex].record.split('')">{{v}}</span>
                    </div>
                  </div>
                </div>
                <div class="right">
                  <div class="time" v-if="indexData && indexData.stop">{{indexData.info || '不能下注'}}<span>{{ _remainingTime || '----'}}</span></div>
                  <div class="time" v-else>{{indexData.info  || '可以下注'}}<span>{{ _remainingTime || '----'}}</span></div>


                  <div class="current">当前期数:<span>{{ indexData.current_period || '----' }}</span>期</div>
                </div>
              </div>
              <div class="row_2"></div>
            </div>

            <!--操作bar-->
            <div class="caozuo">

              <div class="left">
                <el-select v-model="selectedPan" @change="panSelectChange()">
                  <el-option label="A盘" :value="'A'"></el-option>
                  <el-option label="B盘" :value="'B'"></el-option>
                  <el-option label="C盘" :value="'C'"></el-option>
                  <el-option label="D盘" :value="'D'"></el-option>
                </el-select>
              </div>


              <div class="right">


                <!--<div>快速输入：<input type="number" v-model="setAll" @keyup="setAllSelectedCells()"></div>-->
                <!--@keyup="setAllSelectedCells()"-->
                <el-input type="number" placeholder="背景色为黄色的单元格将被同时改变" v-model="setAll">
                  <template slot="append">
                    <el-button @click="setAllSelectedCells()">快速设定</el-button>
                  </template>
                </el-input>

                <el-button @click="reloadWindow()" type="danger"> 重置 </el-button>
                <el-button type="danger">总计：{{_calAllMoney}}</el-button>
                <el-button @click="filterUseFul()" type="success">确定</el-button>
              </div>
            </div>

            <!--投注信息选项-->
            <el-tabs type="border-card" v-model="activeTab" @tab-click="tabsClick">
              <el-tab-pane v-for="tab in pageData" :key="tab.id" :label="tab.name" :name="tab.id">

                <template v-for="(table , i  ) in tab.son">

                  <!-- 表格： 三军 3 / 和值4 -->
                  <div v-if="table.id === '3' || table.id === '4'" class="table">
                    <div class="table-header"> {{table.name}}</div>
                    <div class="table-body">
                      <div class="table-cell" v-for="(cell ,i) in  table.son" :class="{ bgYellow:cell.isSelected}">
                        <span>{{++i}}</span>
                        <span>{{cell.rate}}</span>
                        <span><input v-if="indexData && !indexData.stop"  v-model="cell.val" type="number" @click.self="toggleSelected(cell)"
                                     @blur="checkInput(cell)"></span>
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
                        <span><input v-if="indexData && !indexData.stop" type="number" v-model="cell.val" @click.self="toggleSelected(cell)"
                                     @blur="checkInput(cell)"></span>
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
                        <span class="table-cell"> <span>金额：</span><input v-if="indexData && !indexData.stop" v-model="row.val" type="number"
                                                                         @click.self="toggleSelected(row)"
                                                                         @blur="checkInput(row)"> </span>
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
                        <span><input v-if="indexData && !indexData.stop" type="number" v-model="cell.val" @click.self="toggleSelected(cell)" @blur="checkInput(cell)"></span>
                      </div>
                    </div>
                  </div>


                  <!-- 表格： 欲出几率  181 -->
                  <template v-if="table.id === '181'">
                      <yuchujilv  :items="table.son"></yuchujilv>
                  </template>

                </template>

              </el-tab-pane>
            </el-tabs>


          </template>
          <template v-else>
            {{tabsTip}}
          </template>
        </template>


        <!--开奖结果-->
        <template v-if="selectedMenuIndex === '3'">
          <template v-if="winData">
            <el-table :data="_pagedWinData" stripe style="width: 100%">
              <el-table-column prop="lottery_periods" label="期数" width="180"></el-table-column>
              <el-table-column prop="buy_record" label="购买选项" width="180"></el-table-column>
              <el-table-column prop="put_money" label="购买金额" width="180"></el-table-column>
              <el-table-column prop="get_money" label="获奖金额" width="180"></el-table-column>
              <el-table-column prop="buy_time" label="购买时间" width="180"></el-table-column>
              <el-table-column label="是否中奖" width="180" >
                <template slot-scope="scope">
                  <span  v-if="scope.row.extract > 0" style="color:red">中奖</span>
                  <span v-else> 未中奖 </span>
                </template>
              </el-table-column>


            </el-table>
            <div class="pagination" v-if="winData">
              <el-pagination  :current-page.sync="currentPage" :page-size="pageSize" layout="total, prev, pager, next"
                             :total="winData.length">
              </el-pagination>
            </div>
          </template>
          <tempalte v-else>
            <span>未获取到购买记录</span>
          </tempalte>
        </template>


      </el-main>


    </el-container>


  </el-container>
</template>

<script>
  import Axios from 'axios';
//  import ElButton from "../node_modules/element-ui/packages/button/src/button.vue";
  import yuchujilv from './components/yuchujilv/yuchujilv.vue';

  export default {
    components: { 'yuchujilv': yuchujilv},
    name: 'App',
    data() {
      return {
        tabsTip: '后台数据获取中...',
        setAll: undefined,
        dialogVisible: false,
        api: {
          host:"", //http://jon.linxizhilian.cn
          pageData: "/pagedata",
          buy: "/buyrecord",
          index: "/indexdata",
          user: '/userinfo',
          win:'/winrecord'
        },
        selectedPan: "A",
        activeTab: 'second',
        pageData: null,
        buyData: [],
        winData:undefined,
        pageSize:20,  // 每页的数码
        buyDataMoney: 0,
        userInfo: undefined,
        indexData: undefined, //
        selectedMenuIndex: "2",
        selectedCells: [],
        lotteryIndex: 0,
        Timer: null,
        intervalTimer:null,
        reloadTime: 30000,
        currentPage:1,

      };
    },
    mounted() {
      this.getPageData();  // 获取当前期投注的信息
      this.getIndexData();  // 获取其他信息
      this.getUserInfo();  // 获取用户信息
      this.getWinData();
      console.log( 'mounted')

      const that = this;
      // setTimeout(function () {
      //     that.indexData.stop = true
      // },3000)

    },
    computed: {
      _remainingTime() {
        const that = this;
        const time = that.indexData.remaining_time;
        let M = Math.floor(time / 60);
        let S = parseInt(time % 60);

        if (M < 10) {
          M = `0${M}`
        }
        if (S < 10) {
          S = `0${S}`
        }
        return `${M.toString()}:${S.toString()}`
      },
      _calAllMoney() {
        const that = this;
        let all = 0;
        that.pageData.forEach(function (tab) {
          tab.son.forEach(function (table) {
            table.son.forEach(function (item) {
              let temp = 0;
              if (!isNaN(parseFloat(item.val))) {
                temp = parseFloat(item.val)
              } else {
                temp = 0;
              }
              all += temp
            })
          })
        })
//        console.log( all )
        all = all.toFixed(2)
        that.buyDataMoney = all;
        return all
      },
      _pagedWinData(){
          const that  = this;
          if(!that.winData){
            return
          }

          let start =  (that.currentPage - 1) * that.pageSize;
          let end =start + that.pageSize;


          console.log( start , end )
          return that.winData.slice( start , end )



      },

    },
    methods:{
      setTimer() {
        // 倒计时，在indexData接口数据请求成功并绑定至data后，立即执行定倒计时
        const that = this;


        //剩余时间每一秒减少 1  ，触发_remainingTime 计算属性，并更新到视图
        that.Timer = setInterval(function () {
          if (that.indexData.remaining_time > 0) {
            that.indexData.remaining_time -= 1
          } else {
            // 倒计时到0的时候停止倒计时，并清除定时器，设置remaining_time 为 0
            clearInterval(that.Timer);
            that.indexData.remaining_time = 0
          }
        }, 1000)

        // 保证30秒对indexData数据对更新
        clearInterval( that.intervalTimer )
        that.intervalTimer = setInterval(function () {
          clearInterval(that.Timer);
          that.getIndexData()
        }, that.reloadTime)

      },
      clearTimer() {
        // 清除定时器
        const that = this;
        clearInterval(that.Timer)
      },
      panSelectChange() {
        const that = this;
        if (!that.pageData) {
          return
        }

        if (!that.selectedPan) {
          alert('盘获取错误！！！！')
          return
        }

        that.pageData.forEach(function (tab) {
          tab.son.forEach(function (table) {
            table.son.forEach(function (item) {
              item.pan = that.selectedPan;
//                console.log( item )
            })
          })
        })


      },
      changelotteryIndex(flag) {
        const that = this;
        if (!that.indexData.lottery_data.length) {
          return
        }
        let length = that.indexData.lottery_data.length;

        if (flag > 0) {
          let temp = that.lotteryIndex + 1;
          if (temp >= length) {
            return
          } else {
            that.lotteryIndex = temp
          }
        }
        if (flag < 0) {
          let temp = that.lotteryIndex - 1;
          if (temp < 0) {
            that.lotteryIndex = 0;
          } else {
            that.lotteryIndex = temp
          }
        }
      },
      tabsClick(tab, event) {
        //tab 选项开切换时的调用
//        console.log(tab)


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
        this.clearTimer()

        window.location.reload()
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
        const that = this;
        let isExist = undefined;
        for (let i = 0; i < that.selectedCells.length; i++) {
          if (that.selectedCells[i] === cell) {
            isExist = true
          }
        }
        if (!isExist) {
          that.$set(cell, 'isSelected', true);
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

//        that.checkInput(cell);


//        console.log('selectedCells:', that.selectedCells)

      },

      setAllSelectedCells() {
        // 设置selectedCells.val = setAll
        const that = this;

        if (isNaN(that.setAll) || (that.setAll <= 0)) {
          that.setAll = '';
          alert('请输入正值')
          return
        }

//        console.log('setAll:', that.setAll)

        if (!(that.selectedCells.length > 0)) {
          alert('请先选择需要快速设定的项目，被选择的元素会呈现黄色背景')
          return
        }

        for (let i = 0; i < that.selectedCells.length; i++) {
          let cell = that.selectedCells[i]
          cell.val = that.setAll;
        }

//        console.log('快速输入后 - selectedCells：', that.selectedCells)
      },

//      向后台获取pageData接口数据，并绑定至data.pageData
      getPageData() {
        const that = this;
        Axios.get(`${that.api.host}${that.api.pageData}`).then(
          (res) => {
//            console.log(res.data)
            if(res.data.status === 200){
              res.data.data.forEach(function (tab, i) {
                tab.son.forEach(function (table, i) {

                  if (table.id === '9') {
                    // 红黑码id：9    为数据添加checked 属性为空数组，方便记录checkBox的选择值;
                    table.son.forEach(function (item) {
                      item.val = '';
                      item.checked = [];
                      item.pan = that.selectedPan;
                    })
                  } else {
                    table.son.forEach(function (item) {
                      item.val = '';
                      item.pan = that.selectedPan;

                      // if(item.id == '186'){
                      //   console.log(item)
                      // }
                    })
                  }
                })
              });

              that.pageData = res.data.data;
              that.activeTab = that.pageData[0].id;
            }else{
              if(res.data.status === 300){
                window.location.href = res.data.data;
              }else{
                alert( res.data.message )
              }
            }
          }
        )
      },

      sendDataToBuy() {
        const that = this;

//        检查订单金额是否大于账户余额

        if( parseFloat(that.buyDataMoney ) > parseFloat(that.userInfo.score)){
          alert('余额不足...')
          return
        }

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
//          console.log('success: to buyAPI ', res.data.status)

          if (res.data.status === 200) {
            alert(res.data.message);
            that.dialogVisible = false;
            that.reloadWindow();
          } else {
            if(res.data.status === 300){
              window.location.href = res.data.data;
            }else{
              alert( res.data.message)
              that.reloadWindow();
            }
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
          if(res.data.status === 200){
            that.indexData = res.data.data;
            that.setTimer()  // 设置倒计时器
          }else{
            if(res.data.status === 300){
              window.location.href = res.data.data;
            }else{
              alert( res.data.message)
            }

          }

        }).catch(function (err) {
          console.log('err - getIndexData:', err)
        })
      },
//      向后台请求获取用户信息 post
      getUserInfo() {
        const that = this;
        const url = `${that.api.host}${that.api.user}`;
        const config = {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
        };
        Axios.post(url, config).then(function (res) {
//          console.log('success: to getUserInfo ', res.data)
          if (res.data.status === 200) {
//            console.log('用户信息获取成功：', res.data.message);
            that.userInfo = res.data.data
          } else {
            // alert(res.data.message);
            window.location.href = res.data.data;
          }
        }).catch(function (err) {
          console.log('err: to getUserInfo', err)
        })
      },

      // 获取用户自己的购买开奖结果
      getWinData() {
        const that = this;
        const url = `${that.api.host}${that.api.win}`;
        const config = {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
        };
        Axios.post(url, config).then(function (res) {
//          console.log('success: to getUserInfo ', res.data)
          if (res.data.status === 200) {
//            console.log('开奖结果接口数据获取成功：', res.data);
            that.winData = res.data.data
          } else {
            if(res.data.status === 300){
              window.location.href = res.data.data;
            }else{
              alert( res.data.message)
            }
          }
        }).catch(function (err) {
          console.log('开奖结果接口数据获取失败：', err)
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
        }
        ;

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


        if (!that.buyData.length) {
          return
        }

        that.dialogVisible = true;
//          that.sendDataToBuy()
      },

      handleDelete(index, item) {
//        console.log(index , item);
        const that = this;
        that.buyData[index].val = '';
        that.buyData.splice(index, 1)
      } ,
    },
  }
</script>

<style lang="less">
  * {
    user-select: none;
    box-sizing: border-box;
  }

  #app {

    .bgYellow {
      background: yellow !important;
    }

    width: 100%;
    height: 100%;
    /*background: sandybrown;*/
    .el-header {
      height: 40px !important;
      padding-left: 10px;
      padding-right: 10px;
      padding-top: 8px;
      background-image: url("./assets/headerBg.jpg");
      background-repeat: no-repeat;
      background-size: cover;
      background-position:center center;
      font-size: 16px;
      @color:#fff;

      color: @color;

      @uh:30px;
      .userInfo{
        height: @uh;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        a{
          font-size: 16px;
          color: @color;
          &:hover{
            color: @color;
          }
          &:active{
            color: @color;
          }
        }
        /*background: rgba( 0, 0, 0 ,0.5);*/
        .userInfo-right{
          display: flex;
          flex-direction: row;
          justify-content: flex-end;
          align-items: center;
          .userInfo-name{
            padding-right: 30px;
          }
          .userInfo-score{
            padding-right: 30px;
          }
          .userInfo-logout{
            color:seagreen;
            &:hover{
              text-decoration: underline;
            }
          }
        }

      }

    }

    .innerBox {
      .el-aside {
        padding-top: 30px;
        background: #eeeeee;

        .is-active {
          background: rgba(200, 200, 200, 0.8) !important;
        }

      }
      .el-main {
        /*padding-right: 30px;*/
        padding-top: 5px;
        min-width: 960px !important;
        @media screen and (max-width: 1400px) {
          padding-right: 0;
        }

        @media screen and (min-width: 1401px) {
          padding-right: 15%;
        }


        /*开奖结果中的分页功能视图*/
        .pagination{
            display: flex;
            padding-top: 20px;
            flex-direction: row;
          justify-content: center;
        }


        /*订单弹出层*/
        .dingdanMoney{
          display: flex;
          flex-direction: row;
          padding-top: 20px;
          justify-content: flex-end;
          font-size: 16px;
          >div{
            padding-right: 50px;
          }
        }

        .info {
          display: flex;
          flex-direction: column;
          align-self: center;
          background-image: linear-gradient(to bottom, rgba(177, 110, 37, 0.89), rgba(233, 169, 40, 0.7), rgba(216, 131, 28, 0.94));
          @h: 35px;
          .row_1 {

            height: @h;
            display: flex;
            flex-direction: row;
            justify-content: space-between;
            align-items: center;
            /*background: skyblue;*/
            padding: 3px 5px;
            .left {
              height: 100%;
              display: flex;
              flex-direction: row;

              .ret {
                display: flex;
                flex-direction: row;
                justify-content: flex-start;
                align-items: center;
                font-size: 16px;
                font-weight: bolder;
                .bar {
                  background: silver;
                  display: flex;
                  flex-direction: row;
                  align-self: center;
                  border-radius: 15px;
                  border: 1px solid #a1a1a1;
                  i {
                    display: block;
                    height: 30px;
                    width: 30px;
                    padding: 0 5px;
                    text-align: center;
                    cursor: pointer;
                  }
                  .el-icon-caret-left, .el-icon-caret-right {
                    font-weight: bolder;
                    color: #ff4645;
                    font-size: 20px;
                    position: relative;
                    top: 4px;
                  }
                  span {
                    background: whitesmoke;
                    padding: 0 20px;
                    height: 20px;
                    line-height: 20px;
                    cursor: default;
                    position: relative;
                    top: 5px;
                    color: crimson;
                    font-weight: bolder;
                  }
                }
                .balls {
                  display: flex;
                  align-self: center;
                  padding-left: 5px;
                  .ball {
                    box-sizing: border-box;
                    margin-left: 3px;
                    width: 28px;
                    height: 28px;
                    line-height: 23px;
                    font-size: 16px;
                    color: #ff250f;
                    font-weight: bolder;
                    border-radius: 50%;
                    border: 4px solid #d59322;
                    text-align: center;
                  }
                }
              }
            }
            .right {
              height: @h;
              display: flex;
              flex-direction: row;
              justify-content: flex-end;
              align-self: center;
              line-height: @h;
              .time {
                margin-right: 50px;

                display: flex;
                flex-direction: row;
                align-content: flex-end;
                align-items: center;
                font-size: 16px;
                font-weight: bolder;
                color: #272727;
                span {
                  color: rgb(255, 32, 15);
                  padding: 0 5px;
                }
              }
              .current {
                display: flex;
                flex-direction: row;
                align-content: flex-end;
                align-items: center;
                font-size: 16px;
                font-weight: bolder;
                color: #303133;
                span {
                  color: crimson;
                  padding: 0 5px;
                }
              }
            }

          }
        }

        .caozuo {
          display: flex;
          flex-direction: row;
          justify-content: space-between;
          align-items: center;
          padding-top: 15px;
          padding-bottom: 5px;
          .el-select {
            width: 100px;
            .el-input {
              width: 100%;
            }
          }
          .el-input {
            width: 400px;
            margin-right: 10px;
          }
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
