<template>
  <div 
    class="zy-DragMainBox"
    :style="{
      
    }">
    <div
      class="zy-cardBorderBox"
      v-for="item of data"
      :key="item.id"
      :id="item.id"
      :style="{
        width:cardOutsideWidth+'px', 
        height:cardOutsideHeight+'px'
      }">
      <div
        class="zy-cardInsideBox"
        :style="{ 
            width:cardInsideWidth+'px',
            height:cardInsideHeight+'px'}">
        <div @mousedown="touchStart($event,item.id)" class="zy-topWrapBox">
          <slot name="header" v-bind:item="item">
            <div class="zy-topMenuBox" >
              <div class="zy-menuTitle" v-if="item.name">{{item.name}}</div>
              <div class="zy-menuTitle" v-else> 默认标题 </div>
            </div>
          </slot>
        </div>
        <component :is="item.componentData" :itemData="item" v-if="item.componentData"></component>
        <slot name="content" v-bind:item="item" v-else>
          <div class="zy-emptyContent">
            卡片暂无内容
          </div>
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name:'cardDragger',
  props:{
    data:{
      type:Array,
      default:function(){
        return []
      }
    },
    colNum:{
      type:Number,
      default:2
    },
    cardOutsideWidth:{
      type:Number,
      default:590
    },
    cardOutsideHeight:{
      type:Number,
      default:380
    },
    cardInsideWidth:{
      type:Number,
      default:560      
    },
    cardInsideHeight:{
      type:Number,
      default:320      
    }
  },
  watch:{
    data:{
      handler:function(){
        this.addCardStyle()
      },
      immediate:true
    }
  },
  data: () => ({
    mousedownTimer: null
  }),
  methods:{
    touchStart(event,selectId){

      if(this.mousedownTimer){
        return false;
      }

      let DectetTimer = null;

      const that =this;
      //记录鼠标移动的距离
      let moveLeft = 0,
          moveTop = 0,

      //记录滚动距离
          originTop = document.body.scrollTop === 0 ? document.documentElement.scrollTop : document.body.scrollTop,
          scrolTop = originTop,

      //记录鼠标信息    
          origin_mouse_position ={
            x:0,
            y:0
          },

      //记录组件位置
          origin_obj_position={
            left :0,
            top:0,
            originNum:-1
          },

      //记录交换位置的信息
          OldPosition = null,
          NewPosition = null,

      //选中的卡片的dom和数据
          selectDom = document.getElementById(selectId),
          selectMenuData = this.data.find(item=>{
            return item.id === selectId;
          })

      origin_mouse_position.x = event.screenX;
      origin_mouse_position.y = event.screenY;

      //给选中卡片一个transition：none的class，去除默认过渡
      selectDom.classList.add('zy-moveBox');
      
      //保存现在卡片的top和left
      moveLeft = origin_obj_position.left = parseInt(
         //这里获取到的left是带单位的字符串，要转换成纯数字
        selectDom.style.left.slice(0, selectDom.style.left.length - 2)
      );
      moveTop = origin_obj_position.top = parseInt(
        selectDom.style.top.slice(0, selectDom.style.top.length - 2)
      );

      document.addEventListener('mousemove',mouseMoveListener);
      document.addEventListener('mouseup', mouseUpListener);
      document.addEventListener("scroll", mouseScroll);

      function mouseMoveListener(event){
        moveLeft = origin_obj_position.left + (event.screenX - origin_mouse_position.x);
        moveTop = origin_obj_position.top + (event.screenY - origin_mouse_position.y);
        document.querySelector('.zy-moveBox').style.left = `${moveLeft}px`;
        document.querySelector('.zy-moveBox').style.top = `${moveTop}px`;

       if (!DectetTimer) {
          DectetTimer = setTimeout(()=>{
            cardDetect(moveLeft,moveTop + (scrolTop - originTop)) 
            DectetTimer = null;
          }, 200);
        }    
      }

      function cardDetect(moveItemLeft,moveItemTop){
        //Math.round 四舍五入 Math.round(0.48) == 0
          let newWidthNum = Math.round((moveItemLeft/ that.cardOutsideWidth))+1
        let newHeightNum = Math.round((moveItemTop/ that.cardOutsideHeight))
        if(newHeightNum>(Math.ceil(that.data.length / that.colNum) - 1)||
          newHeightNum<0||
          newWidthNum<=0||
          newWidthNum>that.colNum){
          return false
        }

        const newPositionNum =  (newWidthNum) + newHeightNum * that.colNum;
        console.log(newPositionNum,'newPositionNum');
        if(newPositionNum !== selectMenuData.positionNum){
          let newItem = that.data.find(item =>{
            return item.positionNum === newPositionNum
          })

          if(newItem){
            switchPosition(newItem,selectMenuData);
          }
        }

      }



      function switchPosition(newItem,originItem){
        OldPosition = originItem.positionNum;
        NewPosition = newItem.positionNum;

        if(OldPosition > NewPosition){
          let changeArray = [];
          for(let i = OldPosition -1;i>=NewPosition;i--){
            let pushData = that.data.find(item =>{
              return item.positionNum === i
            })
            changeArray.push(pushData);
          }

          for(let item of changeArray){
            that.$set(item, "positionNum", item.positionNum + 1);
            document.querySelector('#'+item.id).style.top = that.computeTop(item.positionNum)+'px'
            document.querySelector('#'+item.id).style.left = that.computeLeft(item.positionNum) +'px'
          }
         that.$set(originItem, "positionNum", NewPosition);
        }

        if(NewPosition> OldPosition){
          let changeArray=[];
          for(let i=OldPosition+1;i<=NewPosition;i++){
            let pushData = that.data.find(item =>{
              return item.positionNum === i;
            })
            changeArray.push(pushData);
          }
          for (let item of changeArray) {
            that.$set(item, "positionNum", item.positionNum - 1);
            //vue的$set使更改数据的同时实时刷新样式
            document.querySelector('#'+item.id).style.top = that.computeTop(item.positionNum)+'px'
            document.querySelector('#'+item.id).style.left = that.computeLeft(item.positionNum)+'px'
          }
          that.$set(originItem, "positionNum", NewPosition);
        }
      }


      function mouseUpListener(){
         //取消位于交换队列的检测事件、对位置进行最后一次检测
        clearTimeout(DectetTimer)
        DectetTimer = null
        cardDetect(moveLeft,moveTop + (scrolTop - originTop))

        document.querySelector(".zy-moveBox").classList.add('zy-transition');
        document.querySelector(".zy-moveBox").style.top = that.computeTop(selectMenuData.positionNum) + "px";
        document.querySelector(".zy-moveBox").style.left = that.computeLeft(selectMenuData.positionNum) + "px";
        /*mousedownTimer是一个全局定时器，默认为空。详情可看仓库源码。
          若鼠标松开，卡片过渡动画开始时后则激活定时器，
          时间到了的话就清空定时器内容。
          保证在过渡动画执行期间，不能点击其他卡片。
          mousedownTimer在点击事件开始时进行判断，若不为空则直接返回跳出点击事件
        */
        that.mousedownTimer = setTimeout(()=>{
          document.querySelector('.zy-moveBox').classList.remove('zy-transition');
          document.querySelector('.zy-moveBox').classList.remove('zy-moveBox');
          clearTimeout(that.mousedownTimer);
          that.mousedownTimer = null
        },300)

        document.removeEventListener("mousemove", mouseMoveListener);
        document.removeEventListener("mouseup", mouseUpListener);
        document.removeEventListener("scroll", mouseScroll);

      }
      
      function mouseScroll(event) {
        scrolTop =
          document.body.scrollTop === 0
            ? document.documentElement.scrollTop
            : document.body.scrollTop;
        document.querySelector(".d_moveBox").style.top = moveTop + scrolTop - originTop + "px";
      }

    },


    
    /**
     * @description Math.ceil() 函数返回大于或等于一个给定数字的最小整数。
     * @example Math.ceil(7.004) => 8 Math.ceil(-7.004)=>7
     */
    computeTop(num){
      return (Math.ceil(num/this.colNum)-1)* this.cardOutsideHeight;
    },
    computeLeft(num){
      return (num-1)%this.colNum * this.cardOutsideWidth;
    },
    /**
     * @description 给每条项目设置位置
     * 
     */
    addCardStyle(){
      this.$nextTick(()=>{
        this.data.forEach(item=>{
          document.querySelector('#'+item.id).style.top = this.computeTop(item.positionNum)+'px';
          document.querySelector('#'+item.id).style.left = this.computeLeft(item.positionNum) + 'px';
        })
      })
    },

  }
}
</script>

<style>
  .zy-DragMainBox{
    position: relative;
  }

  .zy-cardBorderBox{
    user-select: none;
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.3s;
  }

  .zy-cardInsideBox{
    border-radius: 5px;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  .zy-menuTitle {
    pointer-events: none;
  }
  .zy-topMenuBox {
    height: 50px;
    display: flex;
    align-items: center;
    font-size: 14px;
    color: #838383;
    background-color: white;
    padding: 0px 15px;
  }
  .zy-moveBox {
    top:20px;
    left: 20px;
    z-index: 300;
    transition: none;
  }
  .zy-topWrapBox {
    cursor: move;
    border-bottom: 1px solid #e0e0e0;
  }
  .zy-emptyContent{
    width: 100%;
    height: 100%;
    font-size: 16px;
    color: #979797;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .zy-transition{
    transition: all 0.3s;
  }

</style>