---
layout: post
title: "Custom Alfred iTerm Script"
date:   2025-02-14
tags: [geek]
comments: true
author: xingsong
toc: true
---
# 概述

- 工作中在使用iTerm远程设备，需要打开app，再ssh连接机器，过于麻烦，有没有更方便的方式
- 有的，有的兄弟，那就是Alfred自带的Terminal功能，唯一难点是需要写Applescript

# 实现

参考Github上官方脚本: https://github.com/vitorgalvao/custom-alfred-iterm-scripts

## Applescript 获取

```shell TI:"Copy the script" 
curl --silent 'https://raw.githubusercontent.com/vitorgalvao/custom-alfred-iterm-scripts/master/custom_iterm_script.applescript' | pbcopy
```