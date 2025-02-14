---
layout: post
title: "Custom Alfred iTerm Script"
date:   2025-02-14
tags: [geek]
comments: true
author: xingsong
toc: true
---

工作中在使用iTerm远程设备，需要打开app，再ssh连接机器，过于麻烦，有没有更方便的方式

有的！有的兄弟！那就是Alfred自带的Terminal功能，唯一难点是需要写Applescript

<!-- more -->

# 实现

参考Github上官方脚本: https://github.com/vitorgalvao/custom-alfred-iterm-scripts

## Applescript 获取

```shell
curl --silent 'https://raw.githubusercontent.com/vitorgalvao/custom-alfred-iterm-scripts/master/custom_iterm_script.applescript' | pbcopy
```

## 使用

常规
1. 复制脚本到粘贴板；
2. 打开Alfred Preferences （按win + ,）；
3. 点开Features -> Terminal -> Custom；
4. 把脚本复制到自定义框里

个人
1. 对于我来说我是使用ssh登录的，就需要对脚本内传参的query做点修改；
2. 针对你想要输入的内容进行修改，如果想要输入多次可以用&隔开（具体内容可以看解读）

# Applescript解读

