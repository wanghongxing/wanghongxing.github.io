---
title: 2023 macbook m1 nvm安装node
tags:
---

arm 芯片，用 nvm 安装老版本的node 会提示安装不上，或者提示某个依赖的组件不支持arm64。这时候就需要安装 x64 版本的node。

具体办法就是在访达中找到应用文件夹，右键点击iterm.ap，继续点击显示简介，最后 叉选上 使用Rosetta打开。



然后重新打开命令行，在命令行下重新`nvm install 12.20.12` 就可以了。
