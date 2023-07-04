---
layout: post
title: echarts
date: 2023-5-5 11:17:00 +0800
subtitle: echarts
tags:
  - echarts定制化
categories: 技术
date updated: 2023-05-05 14:14
---

## 动态时间轴数据固定开始结束时间

> 时间轴就是配置项中x轴类型为 `time` 如下：

```js
	option: {
		xAxis: [
			{
				type: 'time', //时间轴配置
				axisLabel: {
					rotate: -45 // 设置刻度标签旋转角度
				}
			}
		]
	}
```

> 如果需要固定开始和结束时间实现动态数据在一个固定范围中增加的效果，可以给x轴增加最大最小值，如下： <mark style="background: #FF5582A6;">注意：</mark> 最大最小值格式必须和data中的时间数据格式一样。

```js
	option: {
		xAxis: [
			{
				type: 'time', //时间轴配置
				min: '2023-5-5 08:00:00',//最小
				max: '2023-5-5 11:10:00',//最大
				axisLabel: {
					rotate: -45 // 设置刻度标签旋转角度
				}
			}
		],
		series: [
			{
				data: [['2023-5-5 08:00:00', 56], [....]]
			}
		]
	}

```

> 整体代码如下：

```js
let base = +new Date(1990, 0, 12);
let oneDay = 24 * 3600 * 1000;
let data = [[base, Math.random() * 300]];

option = {

  tooltip: {
    trigger: 'axis',
    position: function (pt) {
      return [pt[0], '10%'];
    }
  },
  title: {
    left: 'center',
    text: 'Large Ara Chart'
  },
  toolbox: {
    feature: {
      dataZoom: {
        yAxisIndex: 'none'
      },
      restore: {},
      saveAsImage: {}
    }
  },
  xAxis: {
    type: 'time',
    boundaryGap: false,
    min: '1990-1-12',
    max: '1990-1-24'
  },
  yAxis: {
    type: 'value',
    boundaryGap: [0, '100%']
  },
  series: [
    {

      name: 'Fake Data',
      type: 'line',
      smooth: true,
      symbol: 'none',
      areaStyle: {},
      data: data
    }
  ]
};
let timer =  setInterval(function () {
  if(data.length === 10) return  clearInterval(this.timer)
  for (let i = 1; i <= 1; i++) {
    let now = new Date((base += oneDay));
    data.push([+now, Math.round((Math.random() - 0.5) * 20 + data[i - 1][1])]);
  }
  myChart.setOption({
    series: [
      {
        data: data
      }
    ]
  });
}, 1000);

```
