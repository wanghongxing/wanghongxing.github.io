---
title: mark down 的基本用法
date: 2018-11-29 15:01:48
category: fabric
tags:
  - fabric
  - blockchain
---

# 标题1
allala

## 标题二

第二部分

## 无序列表：使用星号、加号或是减号作为列表标记

- Red
- Green
- Blue

## 有序列表：使用数字接着一个英文句点

1. Red
2. Green
3. Blue
> aaa
> bbb

1. 11
2. 333
3. [ ] 不勾选
4. [x] 勾选
5. 223

* [ ] 不勾选
* [x] 勾选


## 强调

在Markdown中，可以使用 * 和  _  来表示斜体和加粗。

## 斜体：
```
*Coding，让开发更简单*
_Coding，让开发更简单_
```
*Coding，让开发更简单*
_Coding，让开发更简单_


## 加粗：

```
**Coding，让开发更简单**
__Coding，让开发更简单__
```

**Coding，让开发更简单**
__Coding，让开发更简单__

## 代码

只要把你的代码块包裹在 “` 之间，你就不需要通过无休止的缩进来标记代码块了。 在围栏式代码块中，你可以指定一个可选的语言标识符，然后我们就可以为它启用语法着色了。 举个例子，这样可以为一段 Ruby 代码着色：

```JS
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```


## 自动链接

方括号显示说明，圆括号内显示网址， Markdown 会自动把它转成链接，例如：
```
[超强大的云开发平台Coding](http://coding.net)
```
[超强大的云开发平台Coding](http://coding.net)


## 表格

在 Markdown 中，可以制作表格，例如：
```
First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell
```


First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell  | Content Cell
Content Cell | Content Cell  | Content Cell


分割线

在 Markdown 中，可以使用 3 个以上『-』符号制作分割线，例如：
```
这是分隔线上部分内容
---
这是分隔线下部分内容
```
效果图如下：

这是分隔线上部分内容
---
这是分隔线下部分内容

没有了