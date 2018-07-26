<template>

  <el-container id="app">
    <el-header>Header</el-header>
    <el-container class="innerBox">
      <el-aside width="260px">
        <div>
          <el-button>默认按钮</el-button>
          <el-button type="primary">主要按钮</el-button>
          <el-button type="success">成功按钮</el-button>
          <el-button type="info">信息按钮</el-button>
          <el-button type="warning">警告按钮</el-button>
          <el-button type="danger">危险按钮</el-button>
        </div>
        <!--<el-collapse v-model="activeNames" @change="handleChange">-->
        <!--<el-collapse-item title="一致性 Consistency" name="1">-->
        <!--<div>与现实生活一致：与现实生活的流程、逻辑保持一致，遵循用户习惯的语言和概念；</div>-->
        <!--<div>在界面中一致：所有的元素和结构需保持一致，比如：设计样式、图标和文本、元素的位置等。</div>-->
        <!--</el-collapse-item>-->
        <!--<el-collapse-item title="反馈 Feedback" name="2">-->
        <!--<div>控制反馈：通过界面样式和交互动效让用户可以清晰的感知自己的操作；</div>-->
        <!--<div>页面反馈：操作后，通过页面元素的变化清晰地展现当前状态。</div>-->
        <!--</el-collapse-item>-->
        <!--<el-collapse-item title="效率 Efficiency" name="3">-->
        <!--<div>简化流程：设计简洁直观的操作流程；</div>-->
        <!--<div>清晰明确：语言表达清晰且表意明确，让用户快速理解进而作出决策；</div>-->
        <!--<div>帮助用户识别：界面简单直白，让用户快速识别而非回忆，减少用户记忆负担。</div>-->
        <!--</el-collapse-item>-->
        <!--<el-collapse-item title="可控 Controllability" name="4">-->
        <!--<div>用户决策：根据场景可给予用户操作建议或安全提示，但不能代替用户进行决策；</div>-->
        <!--<div>结果可控：用户可以自由的进行操作，包括撤销、回退和终止当前操作等。</div>-->
        <!--</el-collapse-item>-->
        <!--</el-collapse>-->


      </el-aside>
      <el-main>
        <template>
          <el-tabs v-model="activeTab" @tab-click="handleClick">

            <el-tab-pane v-for="tab in pageData" :key="tab.id" :label="tab.name" :name="tab.id">
                <div class="table" v-for="(table , i  ) in tab.son" :key="table.id">
                  <div class="table-header"> {{ i }} -- {{table.name}}</div>
                  <div class="table-body">

                      <div class="table-cell" v-for="(row ,i) in  table.son" >

                        <span style="color: red">{{i}}</span> <span>{{row.rate}}</span> <input type="number" v-model="row.val">

                      </div>


                  </div>
                </div>

            </el-tab-pane>
            <!--<el-tab-pane label="配置管理" name="second">配置管理</el-tab-pane>-->
          </el-tabs>
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
        pageData: null

      };
    },
    mounted() {
      var that = this;
      Axios.get(`${that.api.host}${that.api.pageData}`).then(
        (res) => {
          console.log(res.data)
          that.pageData = res.data;
          that.activeTab = that.pageData[0].id;
        }
      )
    },
    methods: {
      handleClick(tab, event) {
        console.log(tab, event);
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
      /*width: 100%;*/
      /*height: 100%;*/
      .el-aside {
        /*width: 100%;*/
        /*height: 100%;*/
        padding-top: 30px;

      }
      .el-main {
        padding-left: 30px;
        .table{
          display: flex;
          flex-direction: column;
          .table-header{
            display: flex;
          }
          .table-body{
            display: flex;
            flex-direction: row;
            flex-wrap: wrap;
            box-sizing: border-box;
            .table-cell{
              border: 1px solid seagreen;
              box-sizing: border-box;
              width: 33%;

            }
          }
        }

      }

    }
  }
</style>
