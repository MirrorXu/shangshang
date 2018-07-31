<template>
  <div class="yuchujilv">
    <template v-if="items.length">
      <div class="item" :style="{width:$itemWidth+'%' }">
        <div class="item-top">
          <span v-if="name1">{{name1 }}</span>
        </div>
        <div class="item-bottom"><span v-if="name2" >{{name2  }}</span></div>
      </div>
      <div class="item" :style="{width:$itemWidth+'%' }"  v-for=" (item , i) in items">
        <div class="item-top">
          <span class="text">{{item.rate || '----'}}</span>
          <span class="zhu" :style="{'height':calHeight(item.rate)}"></span>
        </div>
        <div class="item-bottom">{{item.name || '----'}}</div>
      </div>
    </template>
    <div v-else>
        数据获取中...
    </div>
  </div>
</template>

<script>
	export default {
		name: 'aaa',
    props:
      {
        items:{
          type:Array,
          require:true,
          default:[]
        },
        name1:{
          type:String,
          require:true,
          default:'欲出几率'
        },
        name2:{
          type:String,
          require:true,
          default:'和值'
        },
      },
    data(){
		  return{

      }
    },
    computed:{
      $itemWidth(){
        if(this.items.length > 0){
          return Math.floor((100/this.items.length)*100)/100
        }else{
          return 100
        }
      }
    },
    methods:{
      calHeight(num){
        return Math.ceil( parseFloat(num)*10 )+'px'
      }
    }

	}
</script>

<style lang="less" scope>
    .yuchujilv{
      display: flex;
      flex-direction: row;
      @boderColor:red;
      .item{
        box-sizing: border-box;
        background: #c0c4cc;
        &:nth-child(2n){
          background: #d7dbe4;
        }
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        &-top{
          border-bottom: 1px solid @boderColor;
          display: flex;
          flex-direction: column;
          justify-content: flex-end;
          .text{
            width: 80%;
            text-align: center;
            font-size: 14px;
          }
          .zhu{
            width: 80%;
            background: seagreen;
            height: 40px;
          }

        }
        &-bottom{
          height: 30px !important;
          text-align: center;
          line-height: 30px;
        }
      }
    }

</style>
