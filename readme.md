#### 参数配置

|组件参数：config|说明|
|----|----|
|canMove|是否可滑动|
|size|元素直径(rpx)|
|start|起始位置(0-100 0为12点钟方向)|
|arcLength|弧长(0-100 如：50为半圆)|
|modelValue|滑块当前位置(百分比)|
|bgColor|弧线底部线背景色|
|actColor|弧线活动色|
|dotColor|滑块背景色|
|dotSize|滑块大小(rpx)|
|lineWidth|弧线宽度(rpx)|
|showDot|是否显示滑块|
|showActLine|是否显示动态弧线|

#### 使用示例

    <arc-slider v-model="nowRatio"/>
    const nowRatio = ref<number>(50)

##### 多端支持请自行测试
##### 可扫码进小程序体验 主要针对小程序端实现
##### 有问题咨询本人，不限于创意需求，本插件仅为基础版本
##### 如果对您有帮助 请记得点点star