---
layout: post
title:  "性能优化"
date:   2022-09-14 14:28:51 +0800
subtitle: '性能优化'
tags: 性能
categories: 计算机
---

## 模块化

 >引入的理由就是为了解决文件级别的（分解和聚合导致的）==全局污染== 和 ==依赖混乱== 


#### 主要标准
- AMD
- CMD
- UMD
- CommonJS 
	- ==运行时==
- ESM（Ecmascript Module）
	- ==编译时==：不需要运行代码就能确定依赖关系



## 包管理

>一系列模块的集合

种类：
+ npm：包的属性、registry、cli
+ pnpm
+ yarn


## js工具链

+ 兼容性
	+ API 兼容：
		+ core-js
	+ 语法兼容（syntax transformer runtime）：
		+ regenerator解决 async 和 await问题
		+ tsc 把TypeScript转换为JS
	+ 代码转换集成工具：
		+ babel ：把代码转为抽象语法树==AST==,再把语法树转为代码


## css工具链

> 由于语法的缺失（循环、判断、拼接）、功能的缺失（颜色函数，数学函数，自定义函数）


 
 
 
  