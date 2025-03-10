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

本文基于Github上官方脚本 <[**点击进入Github**]( https://github.com/vitorgalvao/custom-alfred-iterm-scripts )>

## Applescript 获取

```shell
curl --silent 'https://raw.githubusercontent.com/vitorgalvao/custom-alfred-iterm-scripts/master/custom_iterm_script.applescript' | pbcopy
```

## 使用

### 常规
> 1. 执行上述curl命令，会直接复制脚本到粘贴板；
> 2. 打开Alfred Preferences （按 ⌘,）；
> 3. 点开Features -> Terminal -> Custom；
> ![alfred_terminal](/images/2025-02-14-Custom_Alfred_iTerm/alfred_terminal.png)
> 4. 把脚本复制到自定义框里

### 个人
> 1. 对于我来说我是使用ssh登录的，就需要对脚本内传参的query做点修改；
> 2. 针对你想要输入的内容进行修改，如果想要输入多次可以用&隔开（具体内容可以看解读）

# Script解读
<details class="code-box">
  <summary class="code-box-title">
    <span class="summary-arrow"></span>
    <span class="summary-text">AppleScript</span>
    <button class="copy-button">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="16" height="16">
        <path fill="none" d="M0 0h24v24H0z"/>
        <path d="M17 7h-5v-5C12 2.5 11.6 2 11 2H4c-.6 0-1 .5-1 1v16c0 .5.4 1 1 1h16c.5 0 1-.5 1-1V8c0-.6-.5-1-1-1zm-1 14H5V4h6v5h5v12zM8 6h3v3H8V6z"/>
      </svg>
    </button>
  </summary>
<div class="code-box-content">
<pre><span class="line-numbers"></span><code class="code-content">
<span style="color: green;">-- 定义一个变量 是否始终在新窗口中打开 iTerm（主进程中调用，如果为 true 意味着，无论当前 iTerm 中已经有多少窗口或标签页，脚本都会强制在新窗口中打开新的会话）</span>
property open_in_new_window : false

<span style="color: green;">-- 定义一个变量 是否在新标签页中打开 iTerm（主进程中调用，这意味着，新的会话将在当前窗口的新标签页中打开）</span>
property open_in_new_tab : true

<span style="color: green;">-- 定义一个变量 iTerm 是否有开启“quietly”启动选项（即启动时不打开新窗口，后台启动）</span>
property iterm_opens_quietly : false

<span style="color: green;">-- 处理阶段各函数</span>
on new_window()
  <span style="color: green;">-- 创建一个新 iTerm 窗口，使用默认的配置</span>
  tell application "iTerm" to create window with default profile
end new_window 

on new_tab()
  <span style="color: green;">-- 在当前 iTerm 窗口中创建一个新的标签页，使用默认配置</span>
  tell application "iTerm" to tell the first window to create tab with default profile
end new_tab

on call_forward()
  <span style="color: green;">-- 激活 iTerm ，如果 iTerm 存在则页面跳转到应用，如果不存在则打开应用</span>
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
  if not is_running() then return false  <span style="color: green;">-- 判断 iTerm 窗口没有运行，函数返回false</span>

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

<span style="color: green;">-- 主程序</span>
on alfred_script(query)
  if has_windows() then
    <span style="color: green;">-- has_windows函数通过，表示存在窗口；按照开头拟定的规则参数，决定如何处理新的会话</span>
    if open_in_new_window then
      new_window()
    else if open_in_new_tab then
      new_tab()
    else
      <span style="color: green;">-- 复用当前的标签页，可不写</span>
    end if
  else
    <span style="color: green;">-- 该判断防止如果 iTerm 没有窗口，但是设置了后台运行(例如配置了"quietly"启动)，继续执行创建新窗口，会出现两个窗口的情况</span>
    -- one from opening the application, and the other from the command
    <span style="color: green;">-- 当 iTerm 有运行，或开启了“quietly”后台启动，这意味着 iTerm 已经启动，但可能没有窗口，调用new_window函数打开一个新窗口</span>
    if is_running() or iterm_opens_quietly then
      new_window()
    else
      call_forward()  <span style="color: green;">-- iTerm 没有后台运行，用 activate 开启/跳转到应用</span>
    end if
  end if

  <span style="color: green;">-- 输入缓冲处理</span>
  <span style="color: green;">-- macOS 缓冲TTY大小为1024字节，如果 query 超过该大小，会循环检查知道会话不再处理命令，这样做是为了避免输入被截断</span>
  if length of query > 1024
    repeat 500 times <span style="color: green;">-- repeat 循环500次，循环的目的是等待 iTerm 会话完成之前的处理</span>
      if not is_processing() then exit repeat  <span style="color: green;">-- 当 is_processing 函数处理完，返回false，exit repeat 会立刻退出 repeat 循环</span>
      delay 0.01  <span style="color: green;">-- 此处用 delay 而不用 with timeout 是因为 with timeout 和 repeat 不兼容</span>
    end repeat
  end if

  <span style="color: green;">-- 循环等待 iTerm 窗口的出现，并确保向 iTerm 发送文本 query</span>
  repeat 500 times
    if has_windows() then <span style="color: green;">-- 当空窗口出现的时候，传入文本，跳转应用</span>
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
  /* text-align: left; 文本右对齐 */
}

.summary-arrow {
  width: 0;
  height: 0;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-left: 8px solid black;
  margin-right: 5px; /* 添加一些间距 */
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

.copy-button {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
  margin-right: 5px;
}

.copy-button svg {
  vertical-align: middle;
}

.code-box-content pre {
  display: flex; /* 使用 flexbox 布局 */
}

.line-numbers {
  user-select: none; /* 防止用户选择行号 */
  color: #999; /* 行号颜色 */
  padding-right: 10px; /* 行号右侧内边距 */
  text-align: right; /* 行号右对齐 */
}

.code-content {
  flex-grow: 1; /* 代码内容占据剩余空间 */
}
</style>


<script>
  const codeBoxes = document.querySelectorAll('.code-box');

  codeBoxes.forEach(codeBox => {
    const copyButton = codeBox.querySelector('.copy-button');
    const codeContent = codeBox.querySelector('pre code');

    const lineNumbers = codeBox.querySelector('.line-numbers');

    // 生成行号
    const lines = codeContent.textContent.split('\n').length;
    let numbers = '';
    for (let i = 1; i <= lines; i++) {
      numbers += i + '\n';
    }
    lineNumbers.textContent = numbers;

    copyButton.addEventListener('click', () => {
      navigator.clipboard.writeText(codeContent.textContent)
        .then(() => {
          copyButton.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="16" height="16"><path fill="none" d="M0 0h24v24H0z"/><path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/></svg>'; // 替换为对勾图标
          setTimeout(() => {
            copyButton.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="16" height="16"><path fill="none" d="M0 0h24v24H0z"/><path d="M17 7h-5v-5C12 2.5 11.6 2 11 2H4c-.6 0-1 .5-1 1v16c0 .5.4 1 1 1h16c.5 0 1-.5 1-1V8c0-.6-.5-1-1-1zm-1 14H5V4h6v5h5v12zM8 6h3v3H8V6z"/></svg>'; // 恢复复制图标
          }, 2000); // 2秒后恢复
        })
        .catch(err => {
          console.error('复制失败: ', err);
          copyButton.textContent = '复制失败';
          setTimeout(() => {
            copyButton.textContent = '复制';
          }, 2000);
        });
    });
  });
</script>