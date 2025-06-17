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


## 通过tree结构的复选框，实现图标series的删除与添加

> 主要是通过echartsAPI中的echartsInstance内的`setOption` ,具体参考:https://echarts.apache.org/zh/api.html#echartsInstance.setOption

#### 解读api配置：

>设置图表实例的配置项以及数据，万能接口，所有参数和数据的修改都可以通过 `setOption` 完成，ECharts 会合并新的参数和数据，然后刷新图表。如果开启[动画](https://echarts.apache.org/zh/option.html#option.animation)的话，ECharts 找到两组数据之间的差异然后通过合适的动画去表现数据的变化。

#### 调用方式

```js
echartsInstance.setOption(option, opts);
```

**option** : 图表的配置项和数据
**opts** : 其他配置项，包括（notMerge、replaceMerge、lazyUpdate、silent）
+ notMerge : 默认为 *false*(表示合并)  ，是否和之前的*option*合并，
+ replaceMerge：可以指定一个或者多个组件，写法如下
```js
echartsInstance.setOption(option, {
	replaceMerge: ['series', 'xAxis']
});
```
+ lazyUpdate：默认为false，表示同步立即更新，如果为true表示在下一个 animation frame 中才能更新图表
+ silent：默认为false，为true时可以阻止*setOption*抛出事件

#### 组件合并模式
+ 如果设置==`opts.notMerge`==为==`true`==，那么旧的组件会被完全移除，新的组件会根据==`option`==创建。
+ 如果设置==`opts.notMerge`==为==`false`==,或者没有设置：
	+ 如果在==`opts.replaceMerge`==里指定类型，这类组件会进行替换合并
	+ 否则会进行==`普通合并`==
##### 普通合并
	对于一种类型的组件（如：`xAxis`, `series`），新来的 `option` 中的每个“组件描述”（如：`{type: 'xAxis', id: 'xx', name: 'kk', ...}`）会被尽量合并到已存在的组件中。剩余的情况，会在组件列表尾部创建新的组件。
整体规则细节如下：
+ 先依次对==`option`==中每个有声明==`id`==或者==`name`==的组件寻找能匹配其==`id`==或者==`name`==的已有组件，找的则进行合并
+ 再依次对==`option`==中的剩余的组件寻找还未执行过前一条的已有组件，找到的话则合并
+ 最后在==`option`==中剩余的组件，用于创建新的组件
###### 特点：
+ 永远==不会删除==以及存在的组件
+ 组件的索引不会被改变
+ 如果==`id`==和==`name`==没有在==`option`==中被指定，组件会按照它在==`option`==中的顺序合并到已有的组件中。
```js
//已有组件
{ 
	xAxis: [ 
		{ id: 'm', interval: 5 }, 
		{ id: 'n', name: 'nnn', interval: 6 } 
		{ id: 'q', interval: 7 } 
	] 
}
// 新来的 option ：
chart.setOption({ 
	xAxis: [ 
		// id 没有指定。会寻找到第一个没有进行过合并的已有组件，进行合并。 // 即合并到 `id: 'q'`。 
		{ interval: 77 }, 
		// id 没有指定。最终会创建新组件。 
		{ interval: 88 }, 
		// id 没有指定，但是 name 指定了。会被合并到已有的 `name: 'nnn'` 组件。 
		{ name: 'nnn', interval: 66 }, 
		// id 指定了，会被合并到已有的 `id: 'm'` 组件。 
		{ id: 'm', interval: 55 }
	] 
});
//结果组件
{ 
	xAxis: [ 
		{ id: 'm', interval: 55 }, 
		{ id: 'n', name: 'nnn', interval: 66 }, 
		{ id: 'q', interval: 77 }, 
		{ interval: 88 } 
	] 
}
```

##### 替换合并
	对于一种类型的组件（如：`xAxis`, `series`），只有 `option` 中指定了 `id` 并且已有组件中有此 `id` 时，已有组件会和 `option` 相应组件描述进行合并。否则，已有组件都会删除，新组件会被根据 `option` 创建。细节规则如下：
+ 先依次对==`option`==中每个有声明==`id`==组件寻找能匹配其==`id`==的已有组件，找到的话则合并
+ 删除其他没有匹配到
+ 依次对==`option`==中剩余的组件创建新的组件，填入到刚刚删除而空出来的位置上
```js
// 已有组件：
{ 
	xAxis: [ 
	{ id: 'm', interval: 5, min: 1000 }, 
	{ id: 'n', name: 'nnn', interval: 6, min: 1000 },
	{ id: 'q', interval: 7, min: 1000 } 
	] 
}
// 新来的 option :
chart.setOption({ 
	xAxis: [ 
		{ interval: 111 }, 
		// id 已经指定了。因此会被合并进已有的组件 `id: 'q'`。 
		{ id: 'q', interval: 77 }, 
		// id 已经指定了。但是已有组件没有此 id 。 
		{ id: 't', interval: 222 }, { interval: 333 } 
	]
}, { replaceMerge: 'xAxis' });
// 结果组件： 
{ 
	xAxis: [ 
		// 原来的 id 为 'm' 的组件，被移除。 
		// 替换为新的组件。新组件中，并没有原来的 `min: 1000` 了。 
		{ interval: 111 }, 
		// 原来的 id 为 'n' 的组件，被移除。 
		// 替换为新的组件。新组件中，并没有原来的 `min: 1000` 了。 
		{ id: 't', interval: 222 }, 
		// 原来的组件没有被移除，而是和 option 中的组件描述进行了合并。 
		// 所以 `min: 1000` 被保留了。 
		{ id: 'q', interval: 77, min: 1000 }, 
		// 新添加的组件。 
		{ interval: 333 } 
	] 
}
```

##### 删除组件
+ 删除所有：使用==`notMerge: true`==删除所有组件
+ 删除部分：使用==`repalceMerge：[...]`==，被指定 的组件类型，会根据==`replaceMerge`==的规则：如果==`id`==匹配就合并，否则就删除，新组件被创建，有助于保留其他组件的状态（高亮、动画、选中状态等）

##### 举例使用
 > ==a-tree== 配合复选框实现图标series数据的显示和隐藏
 
 利用check事件对象中的==checkedNodesPositions==，代表了当前树形结构中被选中的==节点==，通过选中节点的长度和复选框中的选中的节点长度判断是否选中：
  + ==checkedNodesPositions== 的长度 > ==checkedKeys==的长度 : 表示当前操作是选中节点的操作（包括选中所有节点和选中单个节点）
  + ==checkedNodesPositions== 的长度 < ==checkedKeys==的长度 : 表示当前操作是取消选中节点的操作（包括取消选中所有节点和取消选中单个节点）

##### 代码展示
 
```js
//所选节点列表  
let checkedNodesPositions: CheckInfo['checkedNodesPositions'] = []  
if (e.checkedNodesPositions && e.checkedNodesPositions.length > 0) {  
  //checkedNodesPositions代表当前被选中的节点的数据项  
  checkedNodesPositions = e.checkedNodesPositions  
  
  if (checkedNodesPositions.length > checkedKeys.value.length) {  
    //说明是选中  
    //过滤出多出来的节点  
    const checkedNodes = checkedNodesPositions.filter(  
      (item) => !checkedKeys.value.includes(item.node.id as string),  
    )  
    //再过滤掉父节点  
    const checkedFilterNodes = checkedNodes.filter((item) => item.node.isLeaf)  
    const checkedNodesId = checkedFilterNodes.map(  
      (item) => item.node.id as string,  
    )  
  
    const res = await getEquipmentParamsDataByIdApi(checkedNodesId)  
  
    const lines = res.map((item) => {  
      //处理name相同的问题：如果name相同，则添加前缀,前缀为当前节点父节点  
      return {  
        id: item.id,  
        name:  
          currentNodeName +  
          '_' +  
          checkedFilterNodes.find((n) => n.node.id === item.id)?.node.name +  
          '_' +  
          item.name,  
        type: 'line',  
        smooth: true,  
        showSymbol: false,  
        data: item.data.map((item) => [item.time, item.value]),  
      }  
    })  
    series.push(...lines)  
    lineChart.value?.updateOption({  
      series,  
    })  
  } else {  
    //说明是取消选中  
    //过滤出多出来的节点,进行echart中series的删除  
    //通过指定replaceMerge，实现echarts的替换合并，从而实现复选框取消选择时候的数据删除  
    //具体参考：https://echarts.apache.org/zh/api.html#echartsInstance.setOption  
    const cancelCheckedNodes = checkedNodesPositions.filter((item) =>  
      checkedKeys.value.includes(item.node.id as string),  
    )  
    const cancelCheckedFilterNodes = cancelCheckedNodes.filter(  
      (item) => item.node.isLeaf,  
    )  
    const cancelCheckedNodesId = cancelCheckedFilterNodes.map(  
      (item) => item.node.id as string,  
    )  
    series = series.filter((item) => cancelCheckedNodesId.includes(item.id))  
    lineChart.value?.updateOption(  
      {  
        series: series,  
      },  
      { replaceMerge: ['series'] },  
    )  
  }  
} else {  
  checkedKeys.value = []  
  series = []  
  lineChart.value?.updateOption(  
    {  
      series: series,  
    },  
    { replaceMerge: ['series'] },  
  )  
}  
checkedKeys.value = checkedNodesPositions.map(  
  (item) => item.node.id as string,  
)
```


## vue中使用keep-alive，导致echarts样式（宽度）没有的问题

+ 组件被 keep-alive 缓存时，它的 DOM 结构可能会被隐藏或重新挂载，这可能导致 ECharts 无法正确计算宽度。
+ 可以在echarts重新挂载的钩子函数 ==onActivated（vue为例）== 中对echarts实例重新调用==resize==方法
+ 并且推荐使用==echartInstance.setOption(value)==来进行更新数据，而不是重新使用

