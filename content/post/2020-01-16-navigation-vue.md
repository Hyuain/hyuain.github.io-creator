---
title: "小项目：前端导航页面 - 维生素导航·Vue"
date: 2020-01-16T17:14:39+08:00
draft: false
tags: ["Vue", "前端", "入门", "项目"]
categories: ["从小白开始的前端学习之旅"]
author: "Harvey Zhang"
toc: true
---

初学 Vue，将第一个项目——用 jQuery 和原生 CSS 所做的一个导航页面——给 Renew 了一下。点击可以查看 [代码仓库](https://github.com/Hyuain/navigation-vue) 和 [效果预览](http://hais-teatime.com/navigation-vue)。

<!--more-->

# 功能介绍

目前维生素导航主要分为两个部分：**搜索框** 和 **网页导航**。

在上方的搜索框中输入文字可以使用对应的搜索引擎进行搜索，目前内置三款搜索引擎：Google、百度、Bing，默认为 Google。

点击下方的网页收藏可以进入到对应的页面，目前已经内置了一些默认的页面。

鼠标浮动到图标上会出现删除按钮，点击可以将网页删除；移动设备长按会弹出删除框，点击中间的红色小叉叉也可以删除网页。

点击 **'+'** 会弹出新增网页的对话框，可以输入网页的名称和 URL；若没有输入名称，则将默认使用域名作为名称。然后会自动请求图标，若图标请求失败，将会使用名称的第一个字符作为网页的图标。

网站的信息是存储在 LocalStorage 中的，所以清除历史记录的操作会清空之前添加的网站。

# 在项目中学习

## 搜索框

### 1. 如何实现三个不同搜索引擎的切换？

首先想到的是通过 `v-if` 来实现，设置一个变量 `searchEngine`，点到对应的搜索引擎的时候就将 `searchEngine` 变为对应的值。

那么怎样知道被点击的是哪一个图标呢？好在我们的图标比较少，在 `@click` 的点击事件中，我们可以在 `$event` 之后再传入一个参数，用来表示搜索引擎即可。

这样最简单的切换效果就实现了。

但是我们想再加一点过渡效果，Vue 为我们准备了在 `v-if` 切换出现和消失的时候可以使用的过渡 API。但是简单的使用之后发现一个问题，由于点击切换标签之间实际上是无间隔的，`v-if` 值为 `false` 之后就会马上消失，值为 `true` 的时候又会马上出现，但是动画是需要时间的（进入和退出都有时间），这就导致页面上可能同时出现好几个 LOGO，会变得非常混乱；同时因为 LOGO 在 DOM 中是有上下顺序的，所以按照 DOM 结构从上到下点是正常的，但是如果倒着点就会出现 BUG。

因此，我设计了这样的逻辑：

当我们点击切换搜索引擎时——

1. 立即清除之前的计时器；
2. 立即置空 `searchEngine`（原来的 LOGO 退出 DOM）；
3. 延迟 `400ms` 为 `searchEngine` 赋新值（新的 LOGO 进入 DOM），并记下计时器的 `timerID`。

```js
    changeSearchEngine(e, value) {
      clearTimeout(this.timer)
      this.searchEngine = ''
      this.timer = setTimeout(() => {
        this.searchEngine = value
      }, 400)
    }
```

## 网页列表

### 1. 如何实现移动端的长按效果？

由于网页组件本身就已经比较大了，而且为了通过练习加深对 Vue 的熟悉，我选择将长按效果封装成一个指令来实现，在这里我使用的是全局组件。


```js
// longpress.js
    export default {
      install(Vue, options = {
        time: 1000
      }) {
        Vue.directive('longpress', {
          bind: function(el, binding, vnode, oldVnode) {
          }
        })
      }
    }

// main.js
    import LongPress from '...'
    Vue.use(LongPress, {
      time: 350
    })
```
```xml
<!-- MyComponent.vue-->
    <template>
      <div v-longpress="doSomething"/>
    </template>
```

指令的钩子函数包含 `el`、`binding`、`vnode`、`oldVnode` 这四个参数，指令 `v-longpress` 的主要部分将会用到 `el` 和 `binding`：通过 `el` 我可以获取到指令绑定的元素，也就是现在正在被长按的元素；通过 `binding.value` 我可以获取到指令的绑定值，这里 Vue 会自动识别我的绑定值，我们可以让他绑定值为一个回调函数，在指令中来调用这个函数。

在移动端，我们主要依靠 `touchstart` `touchend` `touchmove` `touchcancel` 来监听触摸状态。

```js
// pressTimer 变量用来存放计时器的 ID
    let pressTimer = null

// longPress 变量用来标记是否是长按
    let longPress = false

// handler 函数用来执行回调函数
    const handler = (e) => {
      binding.value(e, el)
    }

// start 用来启动长按事件：options.time 的时间之后触发 handler
    const start = (e) => {
      e.preventDefault()
      if (e.type === 'click') {
        return
      }
      if (pressTimer === null) {
        pressTimer = setTimeout(() => {
          handler(e)
          longPress = true
        }, options.time)
      }
    }

// cancel 用来取消长按事件：清除计时器
    const cancel = () => {
      if (pressTimer !== null) {
        clearTimeout(pressTimer)
        pressTimer = null
        longPress = false
      }
    }

// 绑定事件
// touchstart 触发长按
    el.addEventListener('touchstart', start)
// touchend 时进行判断，是长按还是短按，短按则触发 click 事件
    el.addEventListener('touchend', (e) => {
      if (pressTimer !== null && longPress === false) {
        el.click()
     }
      cancel(e)
    })
// touchcancel 和 touchmove 时取消长按事件
    el.addEventListener('touchcancel',cancel)
    el.addEventListener('touchmove',cancel)
```

### 2. 如何获取网页的 ICON ？

这一步实际上是在添加网页的对话框组件中实现的，在输入的 URL 进行处理之后，将得到 `https://domain`，先直接在其后面加上 `/favicon.ico` 作为其 ICON。在这之前其实也尝试了一些网络上的 API，但是效果其实都不太好，跟直接用 `/favicon.ico` 获取到的差别不大，并且最终图片也不是很大，不需要清晰度特别高。

然后在网页组件列表组件中直接作为 `<img/>` 的 `src` 尝试获取，若失败再通过 `@error="handelIcoError"` 来调用函数进行处理，替换成文本。

## 添加网页对话框

### 1. 如何呼出对话框？

为了减小组件的逻辑复杂性，我并没有将对话框组件放在网页列表组件之中，而是设计成了第三个平行的组件，当网页列表中的 **'+'** 被点击时，网页列表通过 `eventBus` 将 `addFormOpen` 置为 `true`，从而打开对话框。

我是这样使用 `eventBus` 的：

```js
// App.vue
    import Vue from 'vue'  
    export default {
      data() {
        return {
          eventBus: new Vue({
            data() {
              return {
                addFormOpen: false
              }
            }
          }),
        }
      },
      provide() {
        return {
          eventBus: this.eventBus
        }
      }
    }

// MyComponent.vue
    export default {
      inject: ['eventBus']
    }
```

当对话框确认提交时，他将 `addFormOpen` 置为 `false`，并且将信息通过 `eventBus` 传给网页列表组件。