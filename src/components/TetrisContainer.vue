<template>
  <button id="down" style="display: none;" @click="() => {if(canMove().canMoveDown) toMove('ArrowDown')}">down</button>
  <div class="tetris-container" :style="styleObject">
    <Shape v-for="item in shapes" 
    :key="item"
    :shapeMsg="{
      renderArr: item.renderArr,
      offsetArr: item.offsetArr,
      active: item.active
    }"/>
  </div>
  <div>
    <div style="margin-left: 40px">
      <Next :nextShape="nextShape"/>
    </div>
    <div class="controll" :style="controllStyleObject">
      <div>
        <button @click="reset">重置游戏</button>
      </div>
      <div>
        <span>当前分数:{{grade}}</span>
      </div>
      <div>
        <label for="choice">选择游戏等级</label>
        <select name="选择游戏等级" id="choice" @change="choice" :value="level">
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
          <option value="4">4</option>
          <option value="5">5</option>
          <option value="6">6</option>
          <option value="7">7</option>
          <option value="8">8</option>
          <option value="9">9</option>
        </select>
      </div>
      <div>
        <button @click="start">start</button>
        <button @click="stop">stop</button>
      </div>
      <div class="flexColumnCenter">
        <button @click="rotate()">变形</button>
        <div>
          <button @click="() => {if (canMove().canMoveLeft) toMove('ArrowLeft')}">left</button>
          <button button @click="() => {if (canMove().canMoveRight) toMove('ArrowRight')}">right</button>
        </div>
        <button @click="() => {if (canMove().canMoveDown) toMove('ArrowDown')}">down</button>
        <button button @click="fastDown">fastDown</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import Shape from "./Shape.vue";
import Next from "./Next.vue";
import { PICEBOX_SIZE, GAME_ROWNUM, GAME_COLUMNNUM } from "./../tool/constant";
import shapeArr from "./../tool/shape";
import { onMounted, reactive, ref } from 'vue'
const styleObject = {
  width: PICEBOX_SIZE * GAME_COLUMNNUM + 'px',
  height: PICEBOX_SIZE * GAME_ROWNUM + 'px',
}
const controllStyleObject = {
  height: PICEBOX_SIZE * (GAME_ROWNUM - 5) + 'px',
}
/**
 * renderArr: 用于渲染
 * offsetArr：偏移量,用于实际渲染,renderArr会转换成offsetArr
 * active：是否可以移动变形
 */
const shapes = reactive([])
let level = ref(1) // 游戏等级
let grade = ref(0) // 分数
let clearTotal = ref(0) // 消去的总行数
let timeout = ref()
let starting = ref(false)
let stoping = ref(false)
let nowRandom = ref()
let nextShape = reactive({msg: {}})
const jsonDeepCopy = (obj) => JSON.parse(JSON.stringify(obj))
const getRandomInteger = (min, max) => Math.floor(Math.random() * (max - min + 1) ) + min
const getRandomShape = () => {
  let index = getRandomInteger(0, shapeArr.length - 1)
  nowRandom.value = shapeArr[index]
  return shapeArr[index]
}
shapes.push({renderArr: jsonDeepCopy(getRandomShape()), offsetArr: [], active: true, initWidth: nowRandom.value[0].length})

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

  if (!starting.value) {
    canMoveLeft = false
    canMoveTop = false
    canMoveRight = false
    canMoveDown = false
  }

  let index = getActiveIndex()
  const activeOffsets = shapes[index] && shapes[index].offsetArr
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
  // 未开始游戏,不允许变换
  if (!starting.value) return
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
const deleteByI = (top) => {
  // 删除高度为top的那行
  for (let i = 0; i < shapes.length; i++) {
    if (!shapes[i].active) {
      for (let j = 0; j < shapes[i].offsetArr.length; j++) {
        const item = shapes[i].offsetArr[j]
        if (item.i === top) {
          shapes[i].offsetArr.splice(j, 1)
          j--
        }
      }
    }
  }
  // 删除后比top小的高度全加1
  for (let i = 0; i < shapes.length; i++) {
    for (let j = 0; j < shapes[i].offsetArr.length; j++) {
      const item = shapes[i].offsetArr[j]
      if (item.i <= top) {
        shapes[i].offsetArr[j].i = shapes[i].offsetArr[j].i + 1
      }
    }
  }
}
/**有整行凑齐进行清除 */
const disappear = () => {
  let lockedOffsetArr = getLockedOffsetArr()
  for (let i = 0; i < GAME_ROWNUM; i++) {
    let num = 0
    for (let j = 0; j < lockedOffsetArr.length; j++) {
      const item = lockedOffsetArr[j];
      if (item.i === i) num++
    }
    if (num >= GAME_COLUMNNUM) {
      deleteByI(i)
      clearTotal.value++
      // 累加分数
      grade.value += level.value * 10
      // 判断是否上升游戏等级
      if (clearTotal.value >= 4) {
        if (level.value < 9) {
          level.value++
          clearTotal.value = 0
        }
      }
    }
  }
}
/**方块快速下落 */
const fastDown = () => {
  let fastDownTimeout = setTimeout(function loop() {
    if (canMove().canMoveDown) {
      toMove('ArrowDown')
      loop()
    } else {
      clearTimeout(fastDownTimeout)
    }
  }.bind(this), 50);
}
/**游戏是否结束 */
const ifGameOver = (e) => {
  let arr = getLockedOffsetArr()
  for (let i = 0; i < arr.length; i++) {
    if (arr[i].i === 0) return true
  }
  return false
}
/**选择游戏等级 */
const choice = (e) => {
  level.value = e.target.value
}
const stop = () => {
  clearTimeout(timeout)
  timeout = undefined
  starting.value = false
  stoping.value = true
}
const start = (flag) => {
  // 已经开始游戏,不处理直接返回
  if (starting.value) return
  starting.value = true
  // 随机生成下一个方块数据
  if (!stoping.value) {
    nextShape.msg = {renderArr: jsonDeepCopy(getRandomShape()), offsetArr: [], active: true, initWidth: nowRandom.value[0].length}
  }
  // 获取下落间隔时间
  let time = 1000 - level.value * 100
  if (time <= 100) time = 100
  const BDOM = document.getElementById('down')
  // 设置自动下落
  timeout = setTimeout(function loop() {
    if (canMove().canMoveDown) {
      BDOM.click()
      timeout = setTimeout(loop.bind(this), time);
    } else {
      // 下落到不能移动时候,清除下落的定时器
      clearTimeout(timeout)
      timeout = undefined
      // 固定方块的状态
      shapes[getActiveIndex()].active = false
      // 清除凑齐整行的方块
      disappear()
      // 判断游戏是否结束
      if(ifGameOver()) {
        alert(`游戏结束,您的分数是${grade.value}`)
        return
      }
      // 添加下个出现的方块
      // shapes.unshift({renderArr: jsonDeepCopy(getRandomShape()), offsetArr: [], active: true, initWidth: nowRandom.value[0].length})
      shapes.unshift(nextShape.msg)
      // 处理下个方块的数据
      traversal({
        arr: shapes[0].renderArr,
        index: 0,
        center: true
      })
      // 继续开始自动下落
      starting.value = false
      stoping.value = false
      start()
    }
  }.bind(this), time);
}
const reset = () => {
  shapes.splice(0, shapes.length)
  shapes.push({renderArr: jsonDeepCopy(getRandomShape()), offsetArr: [], active: true, initWidth: nowRandom.value[0].length})
  clearTimeout(timeout)
  timeout = undefined
  starting.value = false
  grade.value = 0
  level.value = 1
  traversal({
    arr: shapes[0].renderArr,
    index: 0,
    center: true
  })
}
onMounted(() => {
  // 初始化游戏等级
  level.value = 1
  // 初始化第一个方块数据
  traversal({
    arr: shapes[0].renderArr,
    index: 0,
    center: true
  })
  // 监听键盘事件 
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
.controll {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.controll button {
  margin: 8px;
}
</style>
