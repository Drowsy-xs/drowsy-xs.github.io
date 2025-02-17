---
layout: post
title: "Alfred调用iTerm"
date:   2025-02-14
tags: [tools]
comments: true
author: xingsong
toc: true
---

问：通常我们用iTerm远程设备，步骤繁琐，首先打开app，再ssh连接机器，过于麻烦，有没有更方便的方式？

答：有的！有的兄弟！那就是Alfred自带的Terminal功能，唯一难点是需要按照你想要的方式撰写Applescript～

<!-- more -->

# 实现

本文基于参考Github上官方脚本 [**点击进入Github**]( https://github.com/vitorgalvao/custom-alfred-iterm-scripts )

## Applescript 获取

```shell
curl --silent 'https://raw.githubusercontent.com/vitorgalvao/custom-alfred-iterm-scripts/master/custom_iterm_script.applescript' | pbcopy
```

## 使用

### 常规
> 1. 复制脚本到粘贴板；
> 2. 打开Alfred Preferences （按win + ,）；
> 3. 点开Features -> Terminal -> Custom；
> 4. 把脚本复制到自定义框里

### 个人
> 1. 对于我来说我是使用ssh登录的，就需要对脚本内传参的query做点修改；
> 2. 针对你想要输入的内容进行修改，如果想要输入多次可以用&隔开（具体内容可以看解读）

# Script解读
<details class="code-box"><summary class="code-box-title"><span class="summary-text">点击打开折叠</span><span class="summary-arrow"></span></summary><div class="code-box-content">
<pre><code>
<span style="color: green;">-- 定义一个变量 控制是否始终在新窗口中打开 iTerm2</span>
property open_in_new_window : false

<span style="color: green;">-- 定义一个变量 控制是否在新标签页中打开 iTerm2</span>
property open_in_new_tab : true

<span style="color: green;">-- 定义一个变量 控制 iTerm2 是否在启动时不打开新窗口（例如，如果后台启动中）</span>
property iterm_opens_quietly : false

<span style="color: green;">-- 处理阶段各函数</span>
on new_window()
  <span style="color: green;">-- 创建一个新 iTerm2 窗口，使用默认的配置</span>
  tell application "iTerm" to create window with default profile
end new_window 

on new_tab()
  <span style="color: green;">-- 在当前 iTerm2 窗口中创建一个新的标签页，使用默认配置</span>
  tell application "iTerm" to tell the first window to create tab with default profile
end new_tab

on call_forward()
  <span style="color: green;">-- 激活 iTerm ，页面跳转到应用</span>
  tell application "iTerm" to activate
end call_forward

<span style="color: green;">-- 假设函数，检查 iTerm 是否运行中</span>
on is_running()
  application "iTerm" is running
end is_running

on is_processing()
  <span style="color: green;">-- 检查当前 iTerm2 会话是否正在处理命令</span>
  tell application "iTerm" to tell the first window to tell current session to return is processing
end is_processing

<span style="color: green;">-- 检查 iTerm2 是否有有效的窗口、标签页和会话。并检查会话中是否存在文本</span>
on has_windows()
  if not is_running() then return false

  tell application "iTerm"  <span style="color: green;">-- tell 块用于将后续命令发送给 iTerm</span>
    if windows is {} then return false  <span style="color: green;">-- 判断 windows 列表是否为空，为空表示没有打开的窗口，函数返回 false</span>
    if tabs of current window is {} then return false  <span style="color: green;">-- 检查标签列表是否为空。如果为空，则表示当前窗口没有标签，函数返回 false</span>
    if sessions of current tab of current window is {} then return false  <span style="color: green;">-- 检查会话列表是否为空。如果为空，则表示当前标签没有会话，函数返回 false</span>

    set session_text to contents of current session of current tab of current window  <span style="color: green;">-- 将获取的会话内容存储在变量 session_text 中</span>
    if words of session_text is {} then return false  <span style="color: green;">-- 检查 session_text 中的单词列表是否为空。如果为空，则表示会话内容为空，函数返回 false</span>
  end tell 

  true  <span style="color: green;">-- 隐式返回true</span>
end has_windows

<span style="color: green;">-- 向当前 iTerm 会话发送文本</span>
on send_text(custom_text) 
  <span style="color: green;">-- 如下按照我的习惯，是先固定命令 ssh 到堡垒机，再通过传入的文本(即设备hostname)连接；按照个人习惯修改 write text 后的参数即可</span>
  tell application "iTerm" to tell the first window to tell current session to write text  "ssh you_name@you_ip\n" & custom_text & return
end send_text

<span style="color: green;">-- 向当前 iTerm 会话发送文本</span>-- 主程序</span>
on alfred_script(query)
  if has_windows() then
    if open_in_new_window then
      new_window()
    else if open_in_new_tab then
      new_tab()
    else
      -- Reuse current tab
    end if
  else
    -- If iTerm is not running and we tell it to create a new window, we get two:
    -- one from opening the application, and the other from the command
    if is_running() or iterm_opens_quietly then
      new_window()
    else
      call_forward()
    end if
  end if

  -- macOS buffers TTY input to 1024 bytes, so if input is larger wait for session to be ready
  -- "with timeout" does not work with "repeat", so use a delay (0.01 * 500 means a timeout of 5 seconds)
  if length of query > 1024
    repeat 500 times
      if not is_processing() then exit repeat
      delay 0.01
    end repeat
  end if

  -- Make sure a window exists before we continue, or the write may fail
  -- "with timeout" does not work with "repeat", so use a delay (0.01 * 500 means a timeout of 5 seconds)
  repeat 500 times
    if has_windows() then
      send_text(query)
      call_forward()
      exit repeat
    end if

    delay 0.01
  end repeat
end alfred_script
    </code></pre>
  </div>
</details>

<style>
.code-box {
  border: 1px solid #ccc; /* 边框 */
  border-radius: 5px; /* 圆角 */
  margin: 10px; /* 外边距 */
  font-family: monospace; /* 等宽字体 */
  background-color: #f9f9f9; /* 背景颜色 */
}

.code-box-title {
  background-color: #f0f0f0; /* 标题背景色 */
  padding: 8px; /* 内边距 */
  cursor: pointer; /* 鼠标样式 */
  font-weight: bold; /* 字体加粗 */
  border-bottom: 1px solid #ccc; /* 标题底部边框 */
  display: flex; /* 使用 flexbox 布局 */
  justify-content: space-between; /* 将内容分散对齐 */
  align-items: center; /* 垂直居中对齐 */
}

.summary-text {
  flex-grow: 1; /* 允许文本扩展以占据剩余空间 */
  text-align: right; /* 文本右对齐 */
}

.summary-arrow {
  width: 0;
  height: 0;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-left: 8px solid black; /* 三角形图标 */
  margin-left: 5px; /* 添加一些间距 */
}

.code-box[open] .summary-arrow {
  transform: rotate(90deg); /* 展开时旋转三角形 */
}

.code-box-content {
  padding: 10px; /* 代码内容内边距 */
  padding-top: 10px;
  padding-bottom: 10px;
  overflow-x: auto; /* 水平滚动条 */
  white-space: pre;
}

/* 如果使用<details>标签，添加以下样式 */
.code-box[open] .code-box-title {
    border-bottom: none;
}
</style>