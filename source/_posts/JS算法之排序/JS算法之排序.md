---
title: JS算法之排序
date: 2018-04-07 20:09:57
tags: JavaScript算法
categories: 数据结构和算法
toc: true
---

来复习一哈各种排序算法，并比较一下优劣, JS实现！

## 排序算法

这可能是最常用和面试最基础的算法了，虽然各种语言都有对应的接口实现，但是最基础的原理我们还是很有必要了解一下的!
在开始之前先介绍两个概念:

1. 大O表示法
	它通常用于描述算法的性能和复杂程度，当讨论大O表示法的时候，一般考虑的是CPU的(时间)占用
	

2. 



### 冒泡排序
	
最容易理解的算法，但性能较低
	
#### 原理

- 冒泡排序通过比较相邻的两个元素，比出大小，然后交换他们的顺序，重复此操作，最后元素达到正确的顺序

#### 实现：

```javascript
var arr = [7, 8, 19, 10, 15]
function bubbleSort(arr = []) {		
	if (Array.isArray(arr)) {
		for(let i = 0; i < arr.length; i++) {
			for(let k = 0; k < arr.length - 1; k++) {
				if(arr[k] > arr[k+1]) // 改成小于号即为从大到小排序
					[arr[k], arr[k+1]] = [arr[k+1], arr[k]]
			}
		}
	}
	return arr;
}
bubbleSort(arr)
	// 输出： 7,8,10,15,19
```
看下图:  ![冒泡过程](JS算法之排序/maopao.png)

- 优化实现: 
通过查看上面的图可以看出,在第一次循环结束后，最后一个已经是最大的了，第二次循环结束后，倒数第二个也已经是正确的数字了，所以最后他们之间的比较是没有必要的，故此算法可以作出如下优化：

```javascript
var arr = [7, 8, 19, 10, 15]
function bubbleSort2(arr = []) {		
	if (Array.isArray(arr)) {
		for(let i = 0; i < arr.length; i++) {
			for(let k = 0; k < arr.length - 1 - i; k++) {  // 减去不需要循环的次数
				if(arr[k] > arr[k+1]) // 改成小于号即为从大到小排序
					[arr[k], arr[k+1]] = [arr[k+1], arr[k]]
			}
		}
	}
	return arr;
}
bubbleSort(arr)
	// 输出： 7,8,10,15,19
```
看下图:  ![冒泡过程2](JS算法之排序/maopao2.png)



### 快速排序












### 选择排序









### 插入排序








### 归并排序











### 堆排序









### 桶排序








## 总结



























