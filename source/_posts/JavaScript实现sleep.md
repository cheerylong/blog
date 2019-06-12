---
title: js实现sleep
date: 2018-3-29 20:42:14
tags: JavaScript
categories: JavaScript
toc: true
---

Java，Shell 等编程语言都存在 `sleep` 函数，用来让线程休眠一段时间，再执行接下来的任务，今天我们来看看 `JavaScript` 中如何实现 `sleep` 呢。众所周知，`JavaScript` 是单线程的，如果让它 `sleep` 一段时间，在浏览器端的表现可能就是假死，卡死，这往往是不合理的。所以这里我们只讨论如何实现 `sleep` 实际情况往往环境提供的定时函数就可以解决问题啦。

## 方案1
	

### 思路

利用 `Date`对象，我们可以获取实时的时间，我们只需要写一个循环，一直比较当前时间和 `sleep` 开始时的时间差值，至到为0就可以啦


### 代码


````javascript

	
	function sleep(time = 0) {
		
		if (!parseFloat(time, 10)) {
			throw new Error('not a number')
		}
		// 随便来个循环
		// let current = Date.now()
		// while(Date.now - current < time)

	    for(let current = Date.now(); Date.now() - current < time;) {}

	}


````


## 方案2


### 思路

利用 `Promise` `async await` `setTimeout` 可以实现类似同步 `sleep` 的体验，但这种方案本质还是跟写一个 `setTimeout` 没有区别，它只是将你想在 `sleep` 后执行的任务一定时间后执行，整个线程不会被堵塞，所以并不是实际的 `sleep`

### 代码

````javascript

function sleep(time = 0) {
    if (!parseFloat(time, 10)) {
        throw new Error('not a number')
    }
    return new Promise((res) => {
        setTimeout(() => {
            res()
        }, time)
    })
}

async function test() {
    await sleep(5000)
    console.log(2)
}
test()
console.log(3)

````


## 总结

上面两种做法，第一种是真正的使线程 `sleep` 也就是执行该方法，所有的其他任务都会被阻塞，而第二种实际可以称为 `setTimeout` 的封装。其他任务并不会受到影响，例如上面的 3 就会先于 2 打印出来。
`JavaScript` 不提供 `sleep` 可能就是因为不需要或者说没有必须使用它的场景，`ES Next`的提出异步编程体验也越来越好，本篇只为思路拓展和记录总结😯。






