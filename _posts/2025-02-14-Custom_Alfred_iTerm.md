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

参考Github上官方脚本: https://github.com/vitorgalvao/custom-alfred-iterm-scripts

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

# Applescript解读
<details class="code-box">
  <summary class="code-box-title">
    <span class="summary-text">点击打开折叠</span>
    <span class="summary-arrow"></span>
  </summary>
  <div class="code-box-content">
    <pre><code>
-- Set this property to true to always open in a new window
property open_in_new_window : false

-- Set this property to false to reuse current tab
property open_in_new_tab : true

-- Set this property to true if iTerm is configured to launch without opening a new window
property iterm_opens_quietly : false

-- Handlers
on new_window()
  tell application "iTerm" to create window with default profile
end new_window 

on new_tab()
  tell application "iTerm" to tell the first window to create tab with default profile
end new_tab

on call_forward()
  tell application "iTerm" to activate
end call_forward

on is_running()
  application "iTerm" is running
end is_running

on is_processing()
  tell application "iTerm" to tell the first window to tell current session to return is processing
end is_processing

on has_windows()
  if not is_running() then return false

  tell application "iTerm"
    if windows is {} then return false
    if tabs of current window is {} then return false
    if sessions of current tab of current window is {} then return false

    set session_text to contents of current session of current tab of current window
    if words of session_text is {} then return false
  end tell

  true
end has_windows

on send_text(custom_text)
  set custom_text to custom_text & "\n"
  tell application "iTerm" to tell the first window to tell current session to write text  "ssh you_name@you_ip\n" & custom_text & return
end send_text

-- Main
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