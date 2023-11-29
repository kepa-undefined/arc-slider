<template>
  <view class="arc-container" :style="{ opacity: dotPosition?.x ? 1 : 0 }" @touchstart.stop="onMouseMove"
    @touchmove.stop="onMouseMove">
    <view class="arc-circle arc-circle-bg" />
    <view class="arc-circle" v-if="showActLine" />
    <view class="arc-dot"
      :style="{ top: `${dotPosition?.y - dotR}px`, left: `${dotPosition?.x - dotR}px`, opacity: showDot ? '1' : '0' }" />
    <view class="custom-box">
      <slot />
    </view>
  </view>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";

/* _____________⬇️rpx转px⬇️_____________ */

const RpxToPx = (rpx: number) => {
  const { screenWidth } = uni.getSystemInfoSync()
  return screenWidth / 750 * rpx
}

/* _____________⬇️参数配置⬇️_____________ */

interface Config {
  canMove?: boolean, // 是否可滑动
  size: number, // 元素直径(rpx)
  start: number, // 起始位置(0-100 0为12点钟方向)
  arcLength: number, // 弧长(0-100 如：50为半圆)
  modelValue: number, // 滑块当前位置(百分比)
  bgColor: string, // 弧线底部线背景色
  actColor: string, // 弧线活动色
  dotColor: string, // 滑块背景色
  dotSize: number, // 滑块大小(rpx)
  lineWidth: number, // 弧线宽度(rpx)
  showDot: boolean, // 是否显示滑块
  showActLine: boolean, // 是否显示动态弧线
}
const props = withDefaults(defineProps<Config>(), {
  canMove: true,
  size: 600,
  start: 75,
  modelValue: 50,
  arcLength: 50,
  bgColor: '#F5F6F7',
  actColor: '#813ACB',
  dotColor: '#FFFF00',
  dotSize: 100,
  lineWidth: 60,
  showDot: true,
  showActLine: true
})
const { size, start, dotSize, arcLength, bgColor, actColor, lineWidth } = props
const startAngle = start / 100 * 360, // 起始角度转换圆心角
  allAngle = arcLength / 100 * 360; // 弧长
const r = RpxToPx(size / 2), // 半径
  dotR = RpxToPx(dotSize / 2), // 滑块半径
  A = { x: r, y: r }, // 圆心坐标
  O = { x: r, y: r * (lineWidth / size) }; // 0度坐标点
const emit = defineEmits(['update:modelValue'])

/* _____________⬇️css bind⬇️_____________ */

const dotStyle = ref(`${RpxToPx(dotSize)}px`)
const boxSize = computed(() => `${RpxToPx(size)}px`)
const lineDot = ref(`${RpxToPx(lineWidth)}px`) // 弧线首尾圆角大小css
const mask = computed(() => { // css mask
  const lineW = `${RpxToPx((size - lineWidth * 2) / 2)}px`
  return `radial-gradient(transparent, transparent ${lineW}, #000 ${lineW}, #000 100%)`
})
const bgLine = computed(() => { // 背景弧线css
  let generate = `conic-gradient(transparent 0, transparent ${start - .1}%, ${bgColor} ${start}%, ${bgColor} ${start + arcLength}%, transparent ${start + arcLength}%)`
  if (arcLength === 100) {
    generate = `conic-gradient(${bgColor} 0, ${bgColor} 100%)`
  } else if (start + arcLength >= 100) {
    const abs = start + arcLength - 100
    generate = `conic-gradient(${bgColor} 0, ${bgColor} ${abs - .1}%,transparent ${abs}%, transparent ${start - .1}%, ${bgColor} ${start}%,${bgColor} 100%)`
  }
  return generate
})
const actLine = computed(() => { // 滑动弧线css
  const nowPos = start + props.modelValue / 100 * arcLength
  let generate = `conic-gradient(transparent 0, transparent ${start - .1}%, ${actColor} ${start}%, ${actColor} ${nowPos}%, transparent ${nowPos}%)`
  if (start + arcLength >= 100) {
    if (nowPos >= 100) {
      const abs = parseInt(`${props.modelValue / 100 * arcLength + start - 100}`)
      generate = `conic-gradient(${actColor} 0, ${actColor} ${abs}%,transparent ${abs + .1}%,transparent ${start - .1}%, ${actColor} ${start}%,${actColor} 100%)`
    }
  }
  return generate
})
const startDot = computed(() => { // 弧线起点圆角位置
  return {
    x: `${calcDotPosition(0).x}px`,
    y: `${calcDotPosition(0).y}px`
  }
})
const endDot = computed(() => { // 背景弧线结束圆角位置
  return {
    x: `${calcDotPosition(100).x}px`,
    y: `${calcDotPosition(100).y}px`
  }
})
const actEndDot = computed(() => { // 当前滑动弧线圆角位置
  return {
    x: `${calcDotPosition(props.modelValue).x}px`,
    y: `${calcDotPosition(props.modelValue).y}px`
  }
})

/* _____________⬇️触摸和滑动事件⬇️_____________ */

const onMouseMove = (e: any) => {
  if (!props.canMove) return
  const { changedTouches } = e
  const x = changedTouches[0].pageX
  const y = changedTouches[0].pageY
  const { top, left } = arcOffset.value
  calcAngle(x - left, y - top)
}

/* _____________⬇️计算当前所在角度⬇️_____________ */

const calcAngle = (x: number, y: number) => {
  const AB = Math.sqrt(Math.pow(A.x - O.x, 2) + Math.pow(A.y - O.y, 2)),
    AC = Math.sqrt(Math.pow(A.x - x, 2) + Math.pow(A.y - y, 2)),
    BC = Math.sqrt(Math.pow(O.x - x, 2) + Math.pow(O.y - y, 2)),
    cosA = (Math.pow(AB, 2) + Math.pow(AC, 2) - Math.pow(BC, 2)) / (2 * AB * AC)

  // 触摸的点到圆心的距离超过范围
  if (AC > r * 1.2 || AC < r * .6) return

  let angleA = Math.round(Math.acos(cosA) * 180 / (Math.PI))
  // 判断夹角在顺时针还是逆时针
  const dx = x - A.x;
  const dy = y - A.y;
  if ((O.x - A.x) * dy - (O.y - A.y) * dx < 0) {
    angleA = 360 - angleA
  }
  const tempAngle = angleA - startAngle < 0 ? Math.round((360 - startAngle + angleA) / allAngle * 100) : Math.round((angleA - startAngle) / allAngle * 100)
  if (tempAngle > 100) return
  emit('update:modelValue', tempAngle)
}

/* _____________⬇️计算滑块所在坐标点⬇️_____________ */

const dotPosition = computed(() => calcDotPosition(props.modelValue))
const calcDotPosition = (num: number) => {
  const cosA = Math.cos((startAngle + allAngle * num / 100) * 2 * Math.PI / 360),
    sinA = Math.sin((startAngle + allAngle * num / 100) * 2 * Math.PI / 360),
    x = (O.x - A.x) * cosA - (O.y - A.y) * sinA + A.x,
    y = (O.y - A.y) * cosA + (O.x - A.x) * sinA + A.y;
  return { x, y }
}


/* _____________⬇️获取元素偏移量⬇️_____________ */

const arcOffset = ref({ top: 0, left: 0 }) // 容器偏移量
// #ifdef MP-WEIXIN
const query = uni.createSelectorQuery().in(getCurrentInstance())
// #endif
// #ifdef APP-PLUS
const query = uni.createSelectorQuery()
// #endif
const init = async () => {
  await query.select('.arc-circle').boundingClientRect((res: any) => {
    const { top, left } = res
    arcOffset.value = { top, left }
  })
  await query.exec()
  await nextTick()
  await calcDotPosition(props.modelValue)
}

init()
</script>

<style lang="scss" scoped>
.arc-container {
  $size: v-bind(boxSize);
  $lineMo: v-bind(lineDot);
  width: $size;
  height: $size;
  margin: auto;
  display: flex;
  position: relative;
  align-items: center;
  justify-content: center;

  .custom-box {
    position: absolute;
    z-index: 2;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .arc-circle {
    $mask: v-bind(mask);
    $startPosX: v-bind("startDot.x");
    $startPosY: v-bind("startDot.y");
    width: $size;
    height: $size;
    mask: $mask;
    position: absolute;
    border-radius: 50%;
    background: v-bind(actLine);

    &::before,
    &::after {
      content: "";
      inset: 0;
      width: $lineMo;
      height: $lineMo;
      border-radius: 50%;
      position: absolute;
      background: v-bind(actColor);
      top: calc($startPosY - $lineMo / 2);
      left: calc($startPosX - $lineMo / 2);
    }

    &::after {
      left: unset;
      $actEndPosX: v-bind("actEndDot.x");
      $actEndPosY: v-bind("actEndDot.y");
      top: calc($actEndPosY - $lineMo / 2);
      left: calc($actEndPosX - $lineMo / 2);
      background: v-bind(actColor);
    }

    &-bg {
      background: v-bind(bgLine);

      &::before,
      &::after {
        inset: 0;
        content: "";
        width: $lineMo;
        height: $lineMo;
        position: absolute;
        border-radius: 50%;
        background: v-bind(bgColor);
        top: calc($startPosY - $lineMo / 2);
        left: calc($startPosX - $lineMo / 2);
      }

      &::after {
        left: unset;
        $endPosX: v-bind("endDot.x");
        $endPosY: v-bind("endDot.y");
        top: calc($endPosY - $lineMo / 2);
        left: calc($endPosX - $lineMo / 2);
      }
    }
  }
}

.arc-dot {
  $dotColor: v-bind("props.dotColor");
  $dotSize: v-bind(dotStyle);
  position: absolute;
  width: $dotSize;
  height: $dotSize;
  border-radius: 50%;
  background: $dotColor ;
  box-shadow: 0 0 10rpx rgba(105, 104, 104, 0.3);
}
</style>