<template>
  <BoxItem v-for="item in offsetArr" 
    :key="item" 
    :top="item.i"
    :left="item.j"
    :active="active"/>
</template>

<script setup>
import BoxItem from "./BoxItem.vue";
import { PICEBOX_SIZE, GAME_ROWNUM, GAME_COLUMNNUM } from "./../tool/constant";
import shapeArr from "./../tool/shape";
import { reactive, ref } from "vue";
const props = defineProps({
  shapeMsg: {
    type: Object,
    default: {}
  }
})
let renderArr = reactive(props.shapeMsg.renderArr) //俄罗斯方块的渲染数据
let offsetArr = reactive(props.shapeMsg.offsetArr) //俄罗斯方块的偏移量数据
let active = ref(props.shapeMsg.active) //俄罗斯方块的是否能移动

console.log(props.shapeMsg);


</script>

<style scoped>
</style>
