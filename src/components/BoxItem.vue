<template>
  <span class="box-item active" :class="classList" :style="styleObject">
  <span class="box-item-inner" :class="classInnerList" :style="styleInnerObject"></span>
  </span>
</template>

<script setup>
import { PICEBOX_SIZE } from "./../tool/constant";
import { reactive, ref, watch } from 'vue'
const props = defineProps({
  top: Number,
  left: Number,
})
const styleObject = reactive({
  width: PICEBOX_SIZE + 'px',
  height: PICEBOX_SIZE + 'px',
  left: props.left * PICEBOX_SIZE + 'px',
  top: props.top * PICEBOX_SIZE + 'px',
})
watch(props, () => {
  styleObject.width = PICEBOX_SIZE + 'px'
  styleObject.height = PICEBOX_SIZE + 'px'
  styleObject.left = props.left * PICEBOX_SIZE + 'px'
  styleObject.top = props.top * PICEBOX_SIZE + 'px'
})
const styleInnerObject = {
  width: PICEBOX_SIZE / 2 + 'px',
  height: PICEBOX_SIZE / 2 + 'px',
  marginLeft: - PICEBOX_SIZE / 2 / 2 + 'px',
  marginTop: - PICEBOX_SIZE / 2 / 2 + 'px',
}
const classList = ['box-item-color6']
const classInnerList = ['box-item-inner-color6']


const count = ref(0)
</script>

<style scoped>
.box-item{
  display: inline-block;
  position: absolute;
}
.box-item-color1 {
  background-color: rgb(93, 190, 190);
}
.box-item-color2 {
  background-color: rgb(184, 33, 126);
}
.box-item-color3 {
  background-color: rgb(214, 99, 16);
}
.box-item-color4 {
  background-color: rgb(68, 203, 15);
}
.box-item-color5 {
  background-color: rgb(86, 6, 78);
}
.box-item-color6 {
  background-color: rgb(231, 234, 25);
}

.box-item-inner {
  display: inline-block;
  position: absolute;
  left: 50%;
  top: 50%;
}
.box-item-inner-color1 {
  background-color: rgb(54, 142, 142);
}
.box-item-inner-color2 {
  background-color: rgb(122, 22, 83);
}
.box-item-inner-color3 {
  background-color: rgb(144, 65, 9);
}
.box-item-inner-color4 {
  background-color: rgb(42, 129, 8);
}
.box-item-inner-color5 {
  background-color: rgb(57, 4, 52);
}
.box-item-inner-color6 {
  background-color: rgb(141, 144, 16);
}
</style>
