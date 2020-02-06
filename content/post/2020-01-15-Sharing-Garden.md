---
title: "小项目：多人共享博客 - Sharing Garden"
date: 2020-01-15T09:57:43+08:00
draft: false
tags: ["Vue", "前端", "入门", "项目"]
categories: ["从小白开始的前端学习之旅"]
author: "Harvey Zhang"
toc: true
---

初学 Vue，一直想给女朋友搞一个博客，不过目前还太会数据库之类的东西，先学习一下多人共享博客的前端页面是如何搭建的，用 Vue 做了一个单页应用。点击可以查看 [代码仓库](https://github.com/Hyuain/kates-blog-client) 和 [效果预览](http://hais-teatime.com/kates-blog-client)。

<!--more-->

# 项目介绍

Sharing Garden 一共有这样几个页面：首页、注册页、登录页、我的页面、用户页面、博客详情、新建博客、编辑博客。

用户可以查看别人的文章、用户信息，或者注册账号之后自己以 markdown 的语法发布或修改文章，也可以查看自己的主页。

项目使用 Vue-cli 进行搭建，通过 Vue-Router 来进行前端页面的跳转，使用 Vuex 存储用户数据，使用 axios 发送请求，选用 element-ui 作为 UI 框架。

# 发送请求

现在大多是使用 axios 来发送 AJAX 请求，本项目也是如此，同时为了再进一步熟悉 axios，我对 axios 文档进行了[翻译](https://hais-teatime.com/hais-notebook/2020/02/02/2020-02-02-Tools-axios/)。原生的 axios 已经能够满足我们的大部分需求了，但是通常我们需要对其进行二次封装来让我们的其他组件使用起来更加方便。

第一层封装：

首先根据前后端约定的基本信息设置 `defaults`，然后统一我们在项目中传入 `config` 的格式，最后统一错误处理方式，因为我们需要处理服务器主动传回的错误和请求本身导致的错误。

```js
    const request = (...) => {
      // ...
      return new Promise((resolve, reject) => {
        axios(config).then(response => {
          if (response.data.status === 'ok') {
            resolve(response.data)
          } else {
            Message.error(response.data.msg)
            reject(response.data)
          }
        }).catch(() => {
          Message.error('网络异常')
          reject({msg: '网络异常'})
        })
      })
    }
```

第二层封装：

服务器端的 API 有博客部分的 `/blog` 和用户部分的 `/auth`，同时我们数据也准备按照这两类来进行处理，所以我们最好对 axios 再进行一层封装，方便以后请求数据。比如在博客部分封装一个获取博客详情的方法：

```js
    const URL = {
      GET_DETAIL: '/blog/:blogId',
    }
    
    const blog = {
      getDetail({blogId}) {
        return request(URL.GET_DETAIL.replace(':blogId', blogId))
      }
    }
    
    // 之后可以这样调用
    blog.getDetail({blogId: anyBlogId}).then(response => {}).catch(error => {})
```

# 页眉和页尾

这次项目是一个简单的单页应用，几个页面的页眉和页尾是差不多的，所以将其抽为 `Header` 和 `Footer` 组件，在这里只提一下 `Header` 组件。

`Header` 在登录和登录的状态下有所不同，因此在加载页面时我们需要对登录状态进行判断，这里我是通过 `dispatch` 一个 `action` 来修改登录状态，然后通过 `getter` 获取到当前的登录状态。

# 注册与登录页面

同样的，注册与登录都是 `dispatch` 了一个 `action`，在这两个 `action` 中，他们都使用之前封装好的 axios API 向服务器发送请求，在拿到结果之后修改用户信息和登录状态。

# 用户和我的页面

这两个页面也是比较类似的，我们可以点击用户头像进入用户页面，点击自己的头像进入我的页面。

在用户页面通过 `$route.params` 获取到 `userId`，然后直接发送请求获取到博客数据和博主的用户数据，这些数据我没有用 Vuex 来存储，因为博客数据更新比较频繁，并且并不需要多个组件共享这个数据，通常只有几个页面需要，而且他们需要的博客数据的内容和形式也不尽相同。

在我的页面就可以直接使用在 `store` 里面存好的 `userInfo`，当然博客数据还是要重新请求。通过删除功能可以向服务器发送请求删除博客，删除完成之后需要在本地更新一下博客列表。使用编辑功能将跳转到编辑页面。

# 创建与编辑页面

在创建博客的页面直接向服务器发送请求，然后获取到创建的结果，并通过 `router.push` 跳转到创建好的博客详情页。编辑页面则要先通过 `$route.params` 获取到当前的 `blogId`，并发送请求获取到博客原来的内容。

# 博客详情页面

博客详情部分主要就是需要将博客内容从 markdown 转换为 HTML 标签，并且给予合适的样式呈现，这里我使用了 `marked`，并且同时使用了 `DOMPurify` 来防止 XSS 攻击。

# 其他问题

另外我们需要关注的一点是，如果我们在没有登录的时候，就不允许访问我的、创建与编辑博客的页面，并且访问上述页面会自动跳转到登录页面，这个时候我们就需要使用到路由元信息和导航守卫。

下面这段代码将在一个导航触发的时候调用，当发现 `meta` 字段中有 `requiresAuth` 之后就会检查登录状态。

```js
    router.beforeEach((to, from, next) => {
      if (to.matched.some(record => record.meta.requiresAuth)) {
        store.dispatch('checkLoginStatus').then(loginStatus => {
          if (!loginStatus) {
            next({
              path: '/login',
              query: {redirect: to.fullPath}
            })
          } else {
            next()
          }
        })
      } else {
        next()
      }
    })
```