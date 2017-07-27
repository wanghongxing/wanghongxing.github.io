---
layout: default
title: markdown语法测试
permalink: /md/
slug: about
lead: "Learn about the project's history, meet the maintaining teams, and find out how to use the Bootstrap brand."
---

# 这是一个测试markdown语法的页面


## whx
wwlsl
## ddd
whx

- 文本1
- 文本2
- 文本3

1. 文本1
2. 文本2
3. 文本3

> 这是演示引用文字的一段

> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。
  <div class="bs-example" data-example-id="badges-in-pills">
    <ul class="nav nav-pills" role="tablist">
      <li role="presentation" class="active"><a href="#">Home <span class="badge">42</span></a></li>
      <li role="presentation"><a href="#">Profile</a></li>
      <li role="presentation"><a href="#">Messages <span class="badge">3</span></a></li>
    </ul>
  </div>

```html
  <div class="bs-example" data-example-id="badges-in-pills">
    <ul class="nav nav-pills" role="tablist">
      <li role="presentation" class="active"><a href="#">Home <span class="badge">42</span></a></li>
      <li role="presentation"><a href="#">Profile</a></li>
      <li role="presentation"><a href="#">Messages <span class="badge">3</span></a></li>
    </ul>
  </div>

```

```javascript
a={
    csslint: {
      options: {
        csslintrc: 'less/.csslintrc'
      },
      dist: [
        'dist/css/bootstrap.css',
        'dist/css/bootstrap-theme.css'
      ],
      examples: [
        'docs/examples/**/*.css'
      ],
      docs: {
        options: {
          ids: false,
          'overqualified-elements': false
        },
        src: 'docs/assets/css/src/docs.css'
      }
    }
};

```