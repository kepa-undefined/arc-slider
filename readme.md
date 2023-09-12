#### 参数配置

|组件参数：config|说明|
|----|----|
|canMove|是否可滑动|
|size|元素大小（基于rpx，最大750）|
|startAngle|起始角度（参考canvas画圆0在3点钟方向）|
|allAngle|整个弧形的长度（0-360）|
|nowRatio|当前滑动比例|
|bgColor|底部线背景色|
|actColor|滑动线背景色|
|dotColor|滑块背景色|
|dotRatio|滑块半径占元素百分比|
|lineRatio|线宽占元素百分比|

####使用示例

    <arc-slider :config="sliderInfo"/>
    const sliderInfo = {
	      canMove: true,
	      size: 600,
	      startAngle: 300,
	      allAngle: 120,
	      nowRatio: 50,
	      bgColor: '#F5F6F7',
	      actColor: '#813ACB',
	      dotColor: '#FFF',
	      dotRatio: 16,
	      lineRatio: 20
    }

#####多端支持请自行测试
#####有问题咨询本人，不限于创意需求，本插件仅为基础版本