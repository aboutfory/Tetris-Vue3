<template>
  <div class="tetris-container" :style="styleObject">
      <Shape v-for="item in shapes" 
        :key="item"
        :shapeMsg="{
          renderArr: item.renderArr,
          offsetArr: item.offsetArr,
          active: item.active
        }"/>
  </div>
</template>

<script setup>
import Shape from "./Shape.vue";
import { PICEBOX_SIZE, GAME_ROWNUM, GAME_COLUMNNUM } from "./../tool/constant";
import shapeArr from "./../tool/shape";
import { onMounted, reactive } from 'vue'
const styleObject = {
  width: PICEBOX_SIZE * GAME_COLUMNNUM + 'px',
  height: PICEBOX_SIZE * GAME_ROWNUM + 'px',
}
/**
 * renderArr: 用于渲染
 * offsetArr：偏移量,用于实际渲染,renderArr会转换成offsetArr
 * active：是否可以移动变形
 */
const shapes = reactive([])
shapes.push({
  renderArr: [[1, 1, 1], [0, 1, 0]],
  offsetArr: [{i: 8, j: 0},{i: 9, j: 0},{i: 10, j: 0},{i: 10, j: 1},],
  active: false
})
shapes.push({renderArr: shapeArr[9], offsetArr: [], active: true})
const traversal = (arr, index, oldTop, oldLeft) => {
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
  /**
   * parseInt(GAME_COLUMNNUM / 2 - ren[ren.length - 1].length / 2)
   * 是为了居中给的初始化水平偏移量
   */
  const off = shapes[index].offsetArr
  const ren = shapes[index].renderArr
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        if (oldTop || oldLeft) {
          off.push({i: i + oldTop, j: j + oldLeft + parseInt(GAME_COLUMNNUM / 2 - ren[ren.length - 1].length / 2)})
        } else {
          off.push({i, j: j + parseInt(GAME_COLUMNNUM / 2 - ren[ren.length - 1].length / 2)})
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
  const activeOffsets = shapes[getActiveIndex()].offsetArr
  const activeRenders = shapes[getActiveIndex()].renderArr
  let tops = [] //所有盒子的高度偏移量(垂直方向)
  let lefts = [] //所有盒子的横向偏移量(水平方向)
  for (let i = 0; i < activeOffsets.length; i++) {
    const el = activeOffsets[i];
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
  if (minleft <= 0) canMoveLeft = false
  if (maxLeft >= GAME_COLUMNNUM - 1) canMoveRight = false
  let actives = document.querySelectorAll('.active')
  let locks = document.querySelectorAll('.lock')
  // console.log({
  //   canMoveLeft,
  //   canMoveTop,
  //   canMoveRight,
  //   canMoveDown,
  // });
  return {
    canMoveLeft,
    canMoveTop,
    canMoveRight,
    canMoveDown,
  }
}
const toMove = (type) => {
  const el = shapes[getActiveIndex()].offsetArr
  for (let i = 0; i < el.length; i++) {
    const item = el[i];
    switch (type) {
      case 'ArrowLeft':
        item.j -= 1
        break;
      case 'ArrowUp':
        item.i -= 1
      break;
      case 'ArrowRight':
        item.j += 1
        break;
      case 'ArrowDown':
        item.i += 1
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
const getActiveIndex = () => {
  // 获取shapes中能够操作移动的Shape组件的index
  for (let i = 0; i < shapes.length; i++) {
    if (shapes[i].active) return i
  }
}
const rotate = () => {
  // 变换前先获取原始偏移量(最小的那个方块的)
  let tops = [] //所有盒子的高度偏移量(垂直方向)
  let lefts = [] //所有盒子的横向偏移量(水平方向)
  for (let i = 0; i < offsetArr.length; i++) {
    const el = offsetArr[i];
    tops.push(el.i)
    lefts.push(el.j)
  }
  // 偏移量升序排序
  tops.sort((a, b) => a - b)
  lefts.sort((a, b) => a - b)
  // 变换俄罗斯方块的形状,实质更改二维数组
  // 如[[1, 0], [1, 0], [1, 1]]顺时针旋转后为[[1, 1, 1], [1, 0, 0]]
  let newArr = [];
  for (let i = 0; i < renderArr[0].length; i++) {
    let temArr = [];
    for (let j = renderArr.length - 1; j >= 0; j--) {
        temArr.push(renderArr[j][i]);
    }
    newArr.push(temArr);
  }
  if (checkIfRotate(newArr, tops[0], lefts[0])) {
    renderArr.splice(0, renderArr,length) //清空已经渲染的方块
    offsetArr.splice(0, offsetArr.length) //清空已经渲染的方块的偏移量
    renderArr = [...newArr]
    traversal(renderArr, getActiveIndex(), tops[0], lefts[0]) //渲染新的
  }
  
}
onMounted(() => {
  traversal(shapes[1].renderArr, 1)


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
