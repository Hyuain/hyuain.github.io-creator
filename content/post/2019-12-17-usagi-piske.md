---
title: "小项目：会动的代码 - Usagi & Piske"
date: 2019-12-17T12:43:58+08:00
draft: false
tags: ["JavaScript","CSS", "前端", "入门", "项目"]
categories: ["从小白开始的前端学习之旅"]
author: "Harvey Zhang"
toc: true
---

初学前端，使用原生 JavaScript 和 CSS 所做的一个小项目。点击可以查看 [代码仓库](https://github.com/Hyuain/usagi-piske-animated) 和 [效果预览](http://hais-teatime.com/usagi-piske-animated)。

<!--more-->

# CSS 部分

## 关于选题

关于练习 CSS 最常见的就是仿写某个页面，但是总感觉这样做让人有些提不起兴趣，毕竟以后工作之后也会写很多页面，不如学习的时候就仿些别的东西？比如某个可爱的小生物？正巧又结识了 CodePen，看到上面有很多有趣的小栗子，着实让人有一试的欲望。于是就选择了 Kanahei 的 Usagi 和 Piske，图片不难又比较可爱，对于我现在的水平来说具有一定的可操作度。

## 在项目中学习

### 1. 如何让屏幕不乱滚？

首先我想要让移动端的屏幕不能随意缩放，因为我会针对移动端做特定的分辨率适配，而且两个小可爱是在屏幕底部的，如果让用户随意缩放，就可能产生一些不必要的错误，因此我得修改 VSCode 默认的 `viewport`，从淘宝手机版抄得 `width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover`。

然后我发现 Usagi 实际上是圆角矩形的一部分，我不能让其他的部分也被用户看到，所以得给 `html` 和 `body` 加上 `overflow:hidden`，同时把它们的 `height` 都设置为 `100%`，如果只有一个设置，有时候可能会出现问题，所以索性两个都加上了。这样屏幕就不会乱滚了。

### 2. Usagi 的耳朵怎么画？

感觉图上的耳朵并不是完全对称的，恍惚间记得之前学习的时候 `border-radius` 有一个加斜线的高级用法，当时搞得不是很清楚，于是又去查了一下，最终使用了比较容易理解的 `border-top-left-radius` `border-top-right-radius` 来分别定义两个圆角，每个属性给予两个值，在控制台调试得到了比较合适的数值。

### 3. Usagi 的耳朵下面为什么有条细线？

仔细观察总感觉 Usagi 的耳朵下面有一条细线，怎么调整他的位置都不会消失，怀疑可能是由于是斜线或者是四舍五入导致的细线，为了美观，为两个耳朵添加伪元素 `::after` 来遮住细线。

### 4. Usagi 的上嘴唇怎么画？

之前学习过一个用 `border` 来画三角形的案例，知道隐藏掉一些 `border` 可能会得到一些特殊的图案。总而言之先尝试隐藏掉两边，再用 `border-radius: 50%` 看看效果，发现效果还比较理想。

### 5. 移动端适配怎么做？

（2020-01-28 更新）

当之前所有的内容做完之后发现一个很大的问题，就是手机上有可能显示不下，因为所有的内容都是用 `px` 写死的，于是就又建了一个 CSS 文件，加上媒体查询，直接用 VSCode 替换将所有的属性在 `500px` 下时全部替换为 `calc(0.7*__px)` （笑）。今天 Renew 这个项目的时候，使用了 rem 方案来做移动端的适配，不过也比较简单，只是以 `500px` 为界设置了不同的 `font-size`，也没有用 JS 来动态改变 `font-size`，因为不希望在大屏设备上的两个小家伙变得太大。

# JavaScript 部分

## 基本思路

思路其实蛮简单，就是将之前写好的 CSS 文件变成字符串，然后将这个字符串同时写到 `<style></style>` 标签和文本中，用计时器使得他一个字符一个字符地写入，同时控制写入的速度。

## 在项目中学习

### 1. 如何一个字一个字地显示？

直接使用 `setInterval`，回调中写 `style.innerHTML` 和 `text.innerHTML` 均为 `styleCode.substring(0, n + 1)` 即可，`n` 从 `0` 开始，每次累加，当 `n >= 0` 时，`clearInterval` 即可。

### 2. 如何显示空格和换行？

当被写入 `text` 之后，空格和换行都被忽略掉了，于是需要对要显示给用户看的 `text` 里面的文本内容进行特殊处理，将 `'\n'` 变为 `'<br>'`，将 `' '` 变为 `'nbsp'`。

### 3. 代码优化

将整个控制组件封装到一个对象 `player` 中，通过 `init` 方法来初始化对象，在初始化时执行 `play` 和 `bindEvents` 方法，开始播放并为播放控件（三档速度调节、播放和暂停按钮）绑定事件。

# 打包

（2020-01-28 更新）

最开始项目使用全局安装的 parcel 进行打包，因此项目没有 `package.json` 文件，项目的入口文件是 `src/index.html`，可以直接使用 parcel 命令来预览和开发。

```bash
parcel src/index.html
```

项目 Renew 后加入了 `deploy.sh` 文件用于打包和上传，

```bash
# 构建
rm -rf dist && parcel build src/index.html --public-url ./ --no-cache --no-minify

# 进入生成的构建文件夹
cd dist

# 初始化 git
git init
git add -A
git commit -m 'deploy'

# 部署到 GitHub Pages
git push -f git@github.com:Hyuain/usagi-piske-animated.git master:gh-pages

cd -
```

parcel 打包出来的 `index.html` 中引用的 CSS 和 JS 文件路径默认是根目录 `'/'`，但是如果部署在了二级目录下面，比如 GitHub Pages 的项目页下面，就会找不到地址，所以需要添加 `--public-url ./` ，将打包后的引用地址改为相对路径。