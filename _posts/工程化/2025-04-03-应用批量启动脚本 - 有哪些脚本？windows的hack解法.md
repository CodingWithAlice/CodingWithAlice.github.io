---
layout:     post
title:     应用批量启动脚本 - 有哪些脚本？windows的hack解法 
subtitle:  
date:       2025-04-03
author:     
header-img: 
catalog: true
tags:
    - < 工作经验沉淀 >
typora-root-url: ..
---



# 应用批量启动脚本 - 有哪些脚本？windows的hack解法

总结：

- 启动脚本根据环境使用不同的语言，包括 shell.sh、script.ps1、batch.bat
- 虽然 windows 无法执行 sh shell.sh，但是 hack 一点，只要安装了 git 就可以使用 git bash 带的 sh 指令了

>  batch 脚本的技术并不能帮助前端能力，但是这个处理的过程、深度了解 windows、linux等系统的差异，可以帮助我更好理解全栈的 "环境无关化" 方向 -> 学习精力应该集中在 **容器化技术 - Docker/Kubernetes**、**云服务集成 - AWS/Serverless**、**跨平台运行时 - Node/Deno/Bun**

- 由于我自己使用 iterm ，可以定制化便于执行 `dev.applescript`



背景：多个应用以 svn 管理，启动时单独启动

目标：节省操作成本

实现方案：①脚本遍历启动应用 + ②package.json 配置指令【第一部分】

### 一、脚本语言选择

最终实现版本 ： shell 脚本 + batch脚本（同时提供）

|                 | 优点                                                         | 缺点                                                         | 备注                                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| shell 脚本      | 非常方便实现，与我而言非常熟悉，3小时左右开发成本            | windows 原生环境下无法执行 sh 指令                           | 安装 git 后 - git bash 自带 sh 指令                          |
| powerShell 脚本 | 1、功能强， 支持面向对象编程、高级函数和复杂的逻辑处理，能够更方便地操作 Windows 系统的各种资源，如注册表、服务、进程等 | 学习曲线陡                                                   | 对于涉及到与系统底层交互或复杂任务处理的情况，PowerShell 更具优势 |
| batch 脚本      | 1、易上手 2、广泛支持：Batch 脚本是 Windows 系统自带的脚本语言，无需额外安装任何软件 | 1、相比 PowerShell，Batch 脚本的功能较为有限，缺乏一些高级的编程特性，处理复杂任务时可能会显得力不从心 | 只适合 windows                                               |

```json
"scripts": {
    "start:main": "npm run start", // 核心应用
    "start:sub-app": "cd sub/sub-app && npm start", // 其他应用
    "start:all": "sh start-apps.sh start", // 一次性启动全部
    "start:custom": "sh start-apps.sh", 
    // 接收参数 npm run start:custom -- -app <app_name * n>
    // 其中的 -- 是为了将 npm 命令和传递给脚本的参数分隔开
}
```

#### 记录各脚本开发过程中遇到的问题

##### 1、shell 脚本

```sh
#!/bin/bash
# 定义启动应用的函数
start_app() {
    case $1 in
        main-app)
            rm -rf src/.umi
            sleep 2 # umi 内部删除会在此应用偶发异常
            echo '正在启动应用 main-app ...'
            npm start # 执行
            if [ $? -ne 0 ]; then
                exit 1
            fi
            ;;
       sub-app)
            rm -rf sub/sub-app/src/.umi
            sleep 2 # umi 内部删除会在此应用偶发异常
            echo '正在启动应用 sub-app ...'
            cd sub/sub-app && NODE_PATH=../node_modules npm start
            if [ $? -ne 0 ]; then
                exit 1
            fi
            ;;
        *)
            exit 1
            ;;
    esac
}
# 检查是否有 -app 参数
if [[ $1 == "-app" ]]; then
    shift  # 移除 -app 这个参数
    start_main=false # 存储是否为核心应用 - 方便匹配工作场景
    other_apps=()
    for app in "$@"; do
        if [[ $app == "main-app" ]]; then
            start_main=true
        else
            other_apps+=("$app")
        fi
    done

    if $start_main; then
        start_app myweb-dva &
        sleep 10 # 等待主应用启动，可根据实际情况调整等待时间
    fi

    for app in "${other_apps[@]}"; do
        start_app "$app" &
    done
    wait  # 等待所有应用启动完成
elif [[ $1 == "start" ]]; then
    start_app main-app &
    sleep 10 # 等待主应用启动，可根据实际情况调整等待时间
    start_app sub-app &
    wait
else
    exit 1
fi
```

##### 2、powerShell 脚本

- 直接找 AI 将上述 shell 脚本转换，但由于 windows 测试时出现很多文件的权限执行问题，比较严格 -> 废弃

##### 3、batch 脚本

- 开麦：一开始偷懒想要直接用 AI 转换的，不想学 batch 语言，觉得价值低，导致 AI 智障，问题反复，现已学得七七八八

记录一些过程中用到的指令和问题

- popd + pushd：先 pushd 保存目录，再 popd 将当前目录切回上次目录 -> 可以保证上下文的一致性
- `exit /b` VS `goto :EOF` VS `call :label`
    - `exit /b` 立即退出整个脚本（推荐场景：严重错误时终止运行）
    - `goto :EOF` 退出当前函数/子程序（推荐场景：函数正常返回）
    - `call :label` 调用子程序（推荐场景：需要获取返回值的场景）
- 阻塞式启动应用
    - `npm start` - 阻塞，卡住脚本，直到这个应用退出才会继续执行代码
    - `call npm start` - 阻塞，阻塞批量处理流程，导致后续代码不执行
    - `start npm start` - 不阻塞，但需要开新窗口(windows 自带的那个 cmd)，在每个窗口执行【不合理】
    - `start \b npm start` - 不阻塞，将应用服务在后台运行，不会打开新窗口【待测试】

```bash
:: 待验证，未投入使用
@echo off
chcp 65001 >nul
setlocal enabledelayedexpansion

if "%~1"=="-app" (
    call :MAIN -app %~2 %~3 %~4
    exit /b
)

::=== 跳转到主程序 ===
goto :MAIN

::定义启动应用的函数
:start_app
set app_name=%~1
echo [INFO] try to start: %app_name%

if "%app_name%"=="main-app" (
    if exist "src\.umi" rmdir /s /q "src\.umi" 2>nul
    ::睡眠4秒
    ping -n 4 127.0.0.1 >nul 
    start /b npm start
    goto :EOF
)

if "%app_name%"=="sub-app" (
    echo [INFO] sub-app start...
    pushd "sub\sub-app"
    start /b cmd /c "set NODE_PATH=..\node_modules && npm start"
    popd
    goto :EOF
)

:MAIN
:: === 主逻辑 ===
if "%~1"=="-app" (
    :: 直接硬编码获取应用参数（跳过-app）
    if "%~2"=="" (
        echo [ERROR] 必须指定至少一个应用
        exit /b 1
    )

    :: 启动应用（不再传递-app参数）
    call :start_app "%~2"
    if not "%~3"=="" (
        ping -n 10 127.0.0.1 >nul
        call :start_app "%~3"
    )
    exit /b
)

:: === 其他命令 ===
if "%~1"=="start" (
    start /b cmd /c "%~f0" :start_app main-app
    ping -n 10 127.0.0.1 >nul
    start /b cmd /c "%~f0" :start_app sub-app
    exit /b
)

exit /b 0
```

