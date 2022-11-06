<template>
  <div class="tetris-container" :style="styleObject">
    <!-- left偏移量前面加上的值是使这些盒子在游戏区域内居中出现 -->
    <BoxItem v-for="item in activeBoxs" 
      :key="item" 
      :top="item.i"
      :left="parseInt(GAME_COLUMNNUM / 2 - 1) + item.j"/>
  </div>
</template>

<script setup>
import BoxItem from "./BoxItem.vue";
import { PICEBOX_SIZE, GAME_ROWNUM, GAME_COLUMNNUM } from "./../tool/constant";
import shapeArr from "./../tool/shape";
import { onMounted, reactive } from 'vue'
const styleObject = {
  width: PICEBOX_SIZE * GAME_COLUMNNUM + 'px',
  height: PICEBOX_SIZE * GAME_ROWNUM + 'px',
}
let messageBoxs = reactive(shapeArr[16]) //俄罗斯方块的渲染数据(每个小方块是否渲染)
let activeBoxs = reactive([]) //俄罗斯方块的偏移量数据
const traversal = (arr, oldTop, oldLeft) => {
  /**
   * 遍历二维数组,第一个循环遍历行
   * 第二个循环遍历行里面的每一项
   * 值为1代表需要创建方块
   * 此时i代表第几行,可用于计算高度偏移值
   * j代表第几列,可用于计算横向偏移值
   */
  /**
   * oldTop为原始垂直偏移量
   * oldLeft为原始水平偏移量
   */
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        if (oldTop || oldLeft) {
          activeBoxs.push({i: i + oldTop, j: j + oldLeft})
        } else {
          activeBoxs.push({i, j})
        }
      }
    }
  }
}
const canMove = () => {
  let canMoveLeft = true
  let canMoveTop = true
  let canMoveRight = true
  let canMoveDown = true
  let tops = [] //所有盒子的高度偏移量(垂直方向)
  let lefts = [] //所有盒子的横向偏移量(水平方向)
  for (let i = 0; i < activeBoxs.length; i++) {
    const el = activeBoxs[i];
    tops.push(el.i)
    lefts.push(el.j)
  }
  // 偏移量升序排序
  tops.sort((a, b) => a - b)
  lefts.sort((a, b) => a - b)
  let minTop = tops[0] //所有盒子中最高处的偏移量
  let maxTop = tops[tops.length - 1] //所有盒子中最低处的偏移量
  let minleft = lefts[0] //所有盒子中最左侧的偏移量
  let maxLeft = lefts[lefts.length - 1] //所有盒子中最右侧的偏移量
  if (minTop <= 0) canMoveTop = false
  if (maxTop >= GAME_ROWNUM - 1) canMoveDown = false
  if (minleft <= 0 - parseInt(GAME_COLUMNNUM / 2 - 1)) canMoveLeft = false
  if (maxLeft > 0 + parseInt(GAME_COLUMNNUM / 2 - 1)) canMoveRight = false
  return {
    canMoveLeft,
    canMoveTop,
    canMoveRight,
    canMoveDown,
  }
}
const toMove = (type) => {
  for (let i = 0; i < activeBoxs.length; i++) {
    const el = activeBoxs[i];
    switch (type) {
      case 'ArrowLeft':
        el.j -= 1
        break;
      case 'ArrowUp':
        el.i -= 1
      break;
      case 'ArrowRight':
        el.j += 1
        break;
      case 'ArrowDown':
        el.i += 1
        break;

      default:
        break;
    }
  }
}
const checkIfRotate = (arr, oldTop, oldLeft) => {
  // 核查变形后是否会超出边界
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        if (oldTop || oldLeft) {
          if (j + oldLeft > parseInt(GAME_COLUMNNUM / 2)) { //超出右边界
            return false
          } else if (i + oldTop > GAME_ROWNUM - 1) { //超出下边界
            return false
          }
        }
      }
    }
  }
  return true
}
const rotate = () => {
  // 变换前先获取原始偏移量(最小的那个方块的)
  let tops = [] //所有盒子的高度偏移量(垂直方向)
  let lefts = [] //所有盒子的横向偏移量(水平方向)
  for (let i = 0; i < activeBoxs.length; i++) {
    const el = activeBoxs[i];
    tops.push(el.i)
    lefts.push(el.j)
  }
  // 偏移量升序排序
  tops.sort((a, b) => a - b)
  lefts.sort((a, b) => a - b)
  // 变换俄罗斯方块的形状,实质更改二维数组
  // 如[[1, 0], [1, 0], [1, 1]]顺时针旋转后为[[1, 1, 1], [1, 0, 0]]
  let newArr = [];
  for (let i = 0; i < messageBoxs[0].length; i++) {
    let temArr = [];
    for (let j = messageBoxs.length - 1; j >= 0; j--) {
        temArr.push(messageBoxs[j][i]);
    }
    // if (temArr && temArr[0]) 
    newArr.push(temArr);
  }
  if (checkIfRotate(newArr, tops[0], lefts[0])) {
    messageBoxs.splice(0, messageBoxs,length) //清空已经渲染的方块
    activeBoxs.splice(0, activeBoxs.length) //清空已经渲染的方块的偏移量
    messageBoxs = newArr
    traversal(messageBoxs, tops[0], lefts[0]) //渲染新的
  }
  
}
onMounted(() => {
  traversal(messageBoxs)
  document.onkeydown = function(event) {
    switch (event.code) {
      case 'ArrowLeft':
        if (canMove().canMoveLeft) {
          toMove('ArrowLeft')
        }
        break;
      case 'ArrowUp':
        rotate()
        break;
      case 'ArrowRight':
        if (canMove().canMoveRight) {
          toMove('ArrowRight')
        }
        break;
      case 'ArrowDown':
        if (canMove().canMoveDown) {
          toMove('ArrowDown')
        }
        break;
      case 'Space':
        
        break;
    
      default:
        break;
    }
  }
})
</script>

<style scoped>
.tetris-container{
  background-color: black;
  position: relative;
}
</style>
