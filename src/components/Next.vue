<template>
  <span>下个方块:</span>
  <div style="position: relative;" :style="nextStyleObject">
    <!-- <div>{{nextShape}}</div> -->
    <Shape :shapeMsg="{
      offsetArr: offsetArr,
      active: false
    }"/>
  </div>
</template>

<script setup>
import Shape from "./Shape.vue";
import { PICEBOX_SIZE } from "./../tool/constant";
import { onMounted, onUpdated, reactive, ref, watch } from 'vue'

const props = defineProps({
  nextShape: {
    type: Object,
    default: {}
  }
})
const nextStyleObject = {
  height: PICEBOX_SIZE * 4 + 'px',
}
let nextShape = reactive(props.nextShape)
let offsetArr = reactive([])
const nextTraversal = (arr) => {
  /**
   * 遍历二维数组,第一个循环遍历行
   * 第二个循环遍历行里面的每一项
   * 值为1代表需要创建方块
   * 此时i代表第几行,可用于计算高度偏移值
   * j代表第几列,可用于计算横向偏移值
   */
  for (let i = 0; i < arr.length; i++) {
    const row = arr[i];
    for (let j = 0; j < row.length; j++) {
      const col = row[j];
      if (col === 1) {
        offsetArr.push({i, j})
      }
    }
  }
}
console.log(nextShape.msg);
// nextTraversal(nextShape.msg.renderArr)
watch(() => nextShape.msg, () => {
  offsetArr.splice(0, offsetArr.length)
  nextTraversal(nextShape.msg.renderArr)
})
</script>

<style scoped>

</style>
