<template>
  <div class="tetris-container" :style="styleObject">
    <button @click="start">start</button>
    <button id="down" style="display: none;" @click="() => {if(canMove().canMoveDown) toMove('ArrowDown')}">down</button>
    <button @click="stop">stop</button>
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
import { onMounted, reactive, ref } from 'vue'
const styleObject = {
  width: PICEBOX_SIZE * GAME_COLUMNNUM + 'px',
  height: PICEBOX_SIZE * GAME_ROWNUM + 'px',
}
/**
 * renderArr: 用于渲染
 * offsetArr：偏移量,用于实际渲染,renderArr会转换成offsetArr
 * active：是否可以移动变形
 */
const jsonDeepCopy = (obj) => JSON.parse(JSON.stringify(obj))
let timeout = ref()
const shapes = reactive([])
shapes.push({renderArr: jsonDeepCopy(shapeArr[22]), offsetArr: [], active: true, initWidth: shapeArr[22][0].length})
shapes.push({
  renderArr: [[1, 1, 1], [0, 1, 0]],
  offsetArr: [{i: 8, j: 0},{i: 9, j: 0},{i: 10, j: 0},{i: 10, j: 1},],
  active: false,
  initWidth: 3
})
shapes.push({
  renderArr: [[1, 1, 1], [0, 1, 1]],
  offsetArr: [{i: 19, j: 7},{i: 18, j: 8},{i: 17, j: 8},{i: 17, j: 8},{i: 19, j: 8}],
  active: false,
  initWidth: 3
})
shapes.push({
  renderArr: [[1, 1, 1], [1, 1, 0]],
  offsetArr: [{i: 3, j: 9},{i: 4, j: 9},{i: 5, j: 9},{i: 6, j: 8},{i: 6, j: 9}],
  active: false
})
shapes.push({
  renderArr: [[1, 1, 1]],
  offsetArr: [{i: 0, j: 8},{i: 0, j: 9},{i: 1, j: 9}],
  active: false
})
shapes.push({
  renderArr: [[1, 1, 1,1,1,1,1,1,1,1,]],
  offsetArr: [{i: 8, j: 0},{i: 8, j: 1},{i: 8, j: 2},{i: 8, j: 3},{i: 8, j: 4},{i: 8, j: 5},{i: 8, j: 6},{i: 8, j: 7},{i: 8, j: 8},{i: 8, j: 9}],
  active: false
})
const traversal = ({arr, index, center, oldTop, oldLeft}) => {
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
   * center代表初始化出现要进行居中
   * parseInt(GAME_COLUMNNUM / 2 - initWidth / 2)
   * 是为了居中给的初始化水平偏移量
   */
  const off = shapes[index].offsetArr
  const initWidth = shapes[index].initWidth
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        if (oldTop || oldLeft) {
          if (center) {
            off.push({i: i + oldTop, j: j + oldLeft + parseInt(GAME_COLUMNNUM / 2 - initWidth / 2)})
          } else {
            off.push({i: i + oldTop, j: j + oldLeft})
          }
        } else {
          if (center) {
            off.push({i, j: j + parseInt(GAME_COLUMNNUM / 2 - initWidth / 2)})
          } else {
            off.push({i, j})
          }
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
  let lockedOffsetArr = getLockedOffsetArr()
  for (let i = 0; i < activeOffsets.length; i++) {
    const item = activeOffsets[i] //每个可以动的小方块
    let ls =[] //小方块左边所有不可以动的小方块的水平偏移量
    let ts =[] //小方块上边所有不可以动的小方块垂直偏移量
    let rs =[] //小方块右边所有不可以动的小方块水平偏移量
    let ds =[] //小方块下边所有不可以动的小方块垂直偏移量
    for (let j = 0; j < lockedOffsetArr.length; j++) {
      const lockedItem = lockedOffsetArr[j] //每个不可以动的小方块
      if (item.i === lockedItem.i) { //同一行
        if (item.j > lockedItem.j) { //同一行左侧
          ls.push(lockedItem.j)
        }
        if (item.j < lockedItem.j) { //同一行右侧
          rs.push(lockedItem.j)
        }
      }
      if (item.j === lockedItem.j) { //同一列
        if (item.i > lockedItem.i) { //同一列上侧
          ts.push(lockedItem.i)
        }
        if (item.i < lockedItem.i) { //同一列下侧
          ds.push(lockedItem.i)
        }
      }
    }
    // 偏移量升序排序
    ls.sort((a, b) => a - b)
    ts.sort((a, b) => a - b)
    rs.sort((a, b) => a - b)
    ds.sort((a, b) => a - b)
    // 判断是否可以左移
    if (ls[ls.length - 1] !== undefined) {
      if (item.j <= ls[ls.length - 1] + 1) {
        canMoveLeft = false
      }
    } else if (item.j <= 0) {
      canMoveLeft = false
    }
    // 判断是否可以上移
    if (ts[ts.length - 1] !== undefined) {
      if (item.i <= ts[ts.length - 1] + 1) {
        canMoveTop = false
      }
    } else if (item.i <= 0) {
      canMoveTop = false
    }
    // 判断是否可以右移
    if (rs[0] !== undefined) {
      if (item.j >= rs[0] - 1) {
        canMoveRight = false
      }
    } else if (item.j >= GAME_COLUMNNUM - 1) {
      canMoveRight = false
    }
    // 判断是否可以下移
    if (ds[0] !== undefined) {
      if (item.i >= ds[0] - 1) {
        canMoveDown = false
      }
    } else if (item.i >= GAME_ROWNUM - 1) {
      canMoveDown = false
    }
  }
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
const checkIfRotate = ({arr, index, center, oldTop, oldLeft}) => {
  // 核查是否可以变形,如果变形后与任意一个锁定的方块位置重合,则不可以变形
  // 如果变形后超出游戏范围,则不可以变形
  const off = []
  const initWidth = shapes[index].initWidth
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        if (oldTop || oldLeft) {
          if (center) {
            off.push({i: i + oldTop, j: j + oldLeft + parseInt(GAME_COLUMNNUM / 2 - initWidth / 2)})
          } else {
            off.push({i: i + oldTop, j: j + oldLeft})
          }
        } else {
          if (center) {
            off.push({i, j: j + parseInt(GAME_COLUMNNUM / 2 - initWidth / 2)})
          } else {
            off.push({i, j})
          }
        }
      }
    }
  }
  let lockedOffsetArr = getLockedOffsetArr()
  for (let i = 0; i < off.length; i++) {
    const ac = off[i];
    if (ac.i < 0 || ac.i >= GAME_ROWNUM) return {ifRotate: false}
    if (ac.j < 0 || ac.j >= GAME_COLUMNNUM) return {ifRotate: false}
    for (let j = 0; j < lockedOffsetArr.length; j++) {
      const lo = lockedOffsetArr[j];
      if (ac.i === lo.i && ac.j === lo.j) return {ifRotate: false}
    }
  }
  return {
    ifRotate: true,
    newOffsetArr: off
  }
}
const getActiveIndex = () => {
  // 获取shapes中能够操作移动的Shape组件的index
  for (let i = 0; i < shapes.length; i++) {
    if (shapes[i].active) return i
  }
}
const getLockedShapes = () => {
  // 获取shapes中不能够操作移动的Shape组件数据
  let result = []
  for (let i = 0; i < shapes.length; i++) {
    if (!shapes[i].active) {
      result.push(shapes[i])
    }
  }
  return result
}
const getLockedOffsetArr = () => {
  // 获取shapes中不能够操作移动的Shape所有组件的全部偏移量数组offsetArr
  let result = []
  let lockedShapes = getLockedShapes()
  for (let i = 0; i < lockedShapes.length; i++) {
    const itemOffsetArr = lockedShapes[i].offsetArr
    for (let j = 0; j < itemOffsetArr.length; j++) {
      result.push(itemOffsetArr[j])
    }
  }
  return result
}
const haveLocked = () => {
  // 是否有下落后被锁定的盒子
  for (let i = 0; i < shapes.length; i++) {
    if (!shapes[i].active) return true
  }
  return false
}
const rotate = () => {
  // 变换前先获取原始偏移量(偏移量最小的那个方块的)
  let tops = [] //所有盒子的高度偏移量(垂直方向)
  let lefts = [] //所有盒子的横向偏移量(水平方向)
  let activeIndex = getActiveIndex()
  let offsetArr = shapes[activeIndex].offsetArr
  let renderArr = shapes[activeIndex].renderArr
  for (let i = 0; i < offsetArr.length; i++) {
    tops.push(offsetArr[i].i)
    lefts.push(offsetArr[i].j)
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
  // 核查是否可以变形
  let check = checkIfRotate({
    arr: newArr, 
    index: activeIndex,
    oldLeft: lefts[0],
    oldTop: tops[0]
  })
  if (check.ifRotate) {
    shapes[activeIndex].renderArr.splice(0, renderArr.length) //清空已经渲染的方块
    shapes[activeIndex].offsetArr.splice(0, offsetArr.length) //清空已经渲染的方块的偏移量
    shapes[activeIndex].renderArr = [...newArr]
    for (let i = 0; i < check.newOffsetArr.length; i++) {
      shapes[activeIndex].offsetArr.push(check.newOffsetArr[i])
    }
    // traversal({
    //   arr: shapes[getActiveIndex()].renderArr, 
    //   index: getActiveIndex(),
    //   oldLeft: lefts[0],
    //   oldTop: tops[0]
    // }) //渲染新的
  }
}
const disappear = () => {
  let lockedOffsetArr = getLockedOffsetArr()
  for (let i = 0; i < GAME_ROWNUM; i++) {
    let num = 0
    for (let j = 0; j < lockedOffsetArr.length; j++) {
      if(lockedOffsetArr[j].i === i) num++
    }
    if (num >= GAME_COLUMNNUM) {
      for (let j = 0; j < lockedOffsetArr.length; j++) {
        if(lockedOffsetArr[j].i === i) {
          delete lockedOffsetArr[j]
          lockedOffsetArr.splice(j, j + 1)
        }
        console.log(lockedOffsetArr);
      }
    }
  }
}
const stop = () => {
  clearTimeout(timeout)
  console.log('stoooooooooooop');
}
const start = () => {
  let leval = localStorage.getItem('leval') || 5
  let time = 1000 - leval * 100
  if (time <= 500) time = 500
  // const event = new UIEvent('keydown', {
  //   view: window,
  //   bubbles: true,
  //   cancelable: true,
  // })
  // event.keycode = 40
  // event.code = 'ArrowDown'
  timeout = setTimeout(function loop() {
    if (canMove().canMoveDown) {
      // document.dispatchEvent(event)
      document.getElementById('down').click()
      setTimeout(loop.bind(this), time);
    } else {
      clearTimeout(timeout)
      shapes[getActiveIndex()].active = false
      disappear()
      shapes.unshift({renderArr: jsonDeepCopy(shapeArr[22]), offsetArr: [], active: true, initWidth: shapeArr[22][0].length})
      traversal({
        arr: shapes[0].renderArr,
        index: 0,
        center: true
      })
      start()
    }
  }.bind(this), time);
}
onMounted(() => {
  traversal({
    arr: shapes[0].renderArr,
    index: 0,
    center: true
  })


  document.onkeydown = function(event) {
    switch (event.code) {
      case 'ArrowLeft':
        if (canMove().canMoveLeft) {
          toMove('ArrowLeft')
        }
        break;
      case 'ArrowUp':
        rotate()
        // if (canMove().canMoveTop) {
        //   toMove('ArrowUp')
        // }
        break;
      case 'ArrowRight':
        if (canMove().canMoveRight) {
          toMove('ArrowRight')
        }
        break;
      // case 'ArrowDown':
      //   if (canMove().canMoveDown) {
      //     toMove('ArrowDown')
      //   }
      //   break;
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
