---
title: 20230221 macbook m1 安装 brew 及 redis 踩坑
tags:
  - redis
---



因为笔记本是用以前的时间仓恢复回来的，所以brew啥的都是x64架构的，需要重新安装brew。

## Homebrew

x86_64 和 ARM64 版本的 homebrew 的安装目录是不一样的

x86_64 安装目录：`/usr/local/homebrew`

ARM64 安装目录：`/opt/homebrew`

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

这是后电脑上有了两套brew,



## 切换命令

为了方便在x64和arm64之间来回切换就参照 [Mac M1 安装 Homebrew 最佳实践](https://wuyanxin.com/post/mac-m1-brew-both-support-aarm64-and-x86_64.html) 做了

文件 `~/.brew_arm`

```
eval "$(/opt/homebrew/bin/brew shellenv)"
```

文件 `~/.brew_intel`

```
eval "$(/usr/local/homebrew/bin/brew shellenv)"
```

将下面代码加入到 `.zshrc`

```
# homebrew
alias brew_arm='source ~/.brew_arm'
alias brew_intel="source ~/.brew_intel"
```

切换命令：

```
brew_intel # 切换到 x86_64
brew_arm # 切换到 arm64
```



## redis

切换到arm64 安装了新的redis `brew install redis`，版本是7.x .

然后再 idea 中调试运行spring boot程序就会卡在中间，打开调试信息看 貌似 redisson 一直不停地连接、close，估计是arm 版本的redis有问题。

先停止redis `brew services stop redis`,然后卸载 `brew uninstall redis`.

切换到 x64环境 brew_intel, 然后重新安装

```
$ brew install redis
$ which redis-server
/usr/local/bin/redis-server   #这就是x64版本

$ brew services start redis

$ file /usr/local/bin/redis-server
/usr/local/bin/redis-server: Mach-O 64-bit executable x86_64 # 确认是x64

```

然后idea中调试就正常了。
