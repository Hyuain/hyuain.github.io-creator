---
title: "小项目：多人共享博客 - Sharing Garden"
date: 2020-01-15T09:57:43+08:00
draft: false
tags: ["Vue", "前端", "入门", "项目"]
categories: ["从小白开始的前端学习之旅"]
author: "Harvey Zhang"
toc: true
---

初学 Vue，一直想给女朋友搞一个博客，不过目前还太会数据库之类的东西，先学习一下多人共享博客的前端页面是如何搭建的，用 Vue 做了一个单页单页应用。点击可以查看 [代码仓库](https://github.com/Hyuain/kates-blog-client) 和 [效果预览](http://hais-teatime.com/kates-blog-client)。

<!--more-->

# 项目介绍

Sharing Garden 一共有这样几个页面：首页、注册页、登录页、我的页面、用户页面、博客详情、新建博客、编辑博客。

项目通过 Vue-Router 来进行前端页面的跳转，用 Vuex 存储用户数据