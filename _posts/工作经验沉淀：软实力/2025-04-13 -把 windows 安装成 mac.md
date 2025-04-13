---
layout:     post
title:     把 windows 安装成 mac
subtitle:  
date:       2025-04-13
author:     
header-img: 
catalog: true
tags:
    - < 工作经验沉淀 >
typora-root-url: ..
---



# 把 windows 安装成 mac

#### 1、按键交换

交换按键的功能 Ctrl → Cmd（Win键）→ Alt 映射成 mac 的 ctrl + option + cmd 功能（ **AutoHotkey**）

```
; ==== 物理按键顺序：Ctrl → Cmd(Win) → Alt → 映射为 Mac：Control → Option → Command ====


; 1. 最左键：Ctrl → 保持为 Control（Mac 的 Control，Windows 原生 Ctrl）
LCtrl::LCtrl  ; 无变化（用于终端快捷键等）


; 2. 中间键：Cmd(Win) → 映射为 Option（Mac 的 Option = Windows 的 Alt）
LWin::LAlt    ; 按 Win 键 → 发送 Alt（模拟 Mac 的 Option）


; 3. 最右键：Alt → 映射为 Command（Mac 的 Command = Windows 的 Ctrl）
LAlt::LCtrl   ; 按 Alt 键 → 发送 Ctrl（模拟 Mac 的 Command）


; ==== 防止误触 ====
; 屏蔽 Win 键单独按下时的开始菜单
~LWin Up::return


; ==== 模拟 Mac 快捷键（Command = 最右键 Alt）====
#If GetKeyState("LAlt", "P")  ; 当按住物理 Alt 键（映射为 Command）时
    c::^c                    ; Command+C → Ctrl+C（复制）
    v::^v                    ; Command+V → Ctrl+V（粘贴）
    z::^z                    ; Command+Z → Ctrl+Z（撤销）
    x::^x                    ; Command+X → Ctrl+X（剪切）
    a::^a                    ; Command+A → Ctrl+A（全选）
    s::^s                    ; Command+S → Ctrl+S（保存）
    q::!F4                   ; Command+Q → Alt+F4（退出应用）
    w::^w                    ; Command+W → Ctrl+W（关闭标签页）
    t::^t                    ; Command+T → Ctrl+T（新建标签页）
    n::^n                    ; Command+N → Ctrl+N（新建窗口）
    f::^f                    ; Command+F → Ctrl+F（查找）
    Space::^Space            ; Command+Space → Ctrl+Space（切换输入法）
#If
```

#### 2、插件安装

- 日历清单（手动同步 todo List）
- 印象笔记（自动同步）
- Listary（保存剪切板的所有历史内容）
- Snipaste（截图置顶功能）
- ConEmu（模拟 ITems2，可以设置快捷键模拟）
