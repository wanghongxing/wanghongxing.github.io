
   <h1 id="create-project" class="page-header">新建项目</h1>


### 新建项目

- 新建目录 demo
- 下载简单的项目模版
[项目模版](http://gitlab.my118.com/static/wghl-template/repository/archive.zip?ref=master)

然后解压缩到指定目录 ，这时候就可以选择用IDE 环境打开项目

- 修改项目名称


修改package.json 中的项目名称
修改server.js中的项目名称
修改pom.xml中的项目名称


- 安装项目所有依赖

如果是使用VSCode，运行任务，windows: Ctrl+Shift+P / Mac : Command + Shift + P ,然后输入tasks 然后选择 运行任务 ,继续选择 npm install 
也可以在终端命令行窗口执行 npm install 


- 运行看看效果
如果是使用VSCode，运行任务，快捷键 windows: Ctrl+Shift+P / Mac : Command + Shift + P ,然后输入**tasks** 然后选择 **运行任务** ,继续选择 npm start 就可以运行起来 
命令行的话输入node server.js来运行程序。
这是后打开浏览器输入 http://localhost:8080/ 就可以查看模版提供的简单页面

- 试着修改一下页面
打开src/views/index.html 试着修改一下内容，然后在浏览器里看一下
打开src/scripts/page/index.js src/styles/page/index.less 试着修改一下，这时候不用刷新脚本你就能看到页面产生了变化，真的蛮神奇的。

其实开发的过程中，你一直保持任务运行状态就行，不用停止刚才启动的任务，你只要更改了页面的脚本和css，浏览器直接就能刷新，目前修改html页面不会刷新。

- 停止运行
如果是使用VSCode，运行任务，快捷键 windows: Ctrl+Shift+P / Mac : Command + Shift + P ,然后输入**tasks** 然后选择 **终止任务** ,就可以停止当前的任务 
命令行的话输入node server.js来运行程序。
这是后打开浏览器输入 http://localhost:8080/ 就可以查看模版提供的简单页面



### 项目最终的目录架构如下：

```
- project

├── webpack.config.js   # webpack的配置文件
├── config.js  #项目独有的配置文件，确定你明白你改的是什么就行
├── package.json  #nodejs开发用的配置文件。
├── pom.xml  #maven项目打包用的配置文件
├── server.js   #开发程序入口
├── src     #前端源码开发目录
│   ├── imgs #图片资源
│   │   ├── favicon.ico
│   ├── json     #模拟数据，所有页面、api需要的模拟数据放到这里，不要把前端真正需要的数据放到这里。
│   │   ├── about.json
│   ├── routes     #express项目路由，这里是前端用nodejs模拟所有的页面、api的地方
│   │   └── index.js
│   ├── scripts   #JS脚本，按照page、components进行组织
│   │   ├── components #这里可以放第三方的js 控件，但是引用方式必须是在页面js中require进来，同名的js css最好放到一起，如果还有响应的图片，那就真个目录放置到这里
│   │   │   ├── jquery-ui.js
│   │   │   ├── jquery.typeahead.css
│   │   │   └── jquery.typeahead.min.js
│   │   ├── page    #这里放每个页面专用的js 文件，会自动嵌入到同名的html文件中
│   │   │   ├── about.js
│   │   │   └── ***.js
│   │   └── parameter  #这里可以放js中使用的参数文件，通过require方式引入，后端模拟数据不要放到这里，这里是前端页面实际需要的数据。
│   │       └── ***.json
│   ├── styles #css目录，按照页面（模块）、通用、第三方三个级别进行组织
│   │   ├── common    #通用的css文件
│   │   │   ├── common.less
│   │   │   ├── global.css
│   │   │   ├── grid.css
│   │   │   ├── main.css
│   │   │   └── reset.css
│   │   ├── lib  #这里可以放第三方的css文件
│   │   └── page #这里放每个页面独有的css文件，，会自动嵌入到同名的html文件中
│   │       ├── export.css
│   │       └── ***.css
│   └── views      #HTML模板，每个页面放一个html文件
│       ├── error.html
│       ├── index.html
│       └── ***.html
├── target #项目打包后的输出目录
│   ├── static #最终打包生成的静态资源目录，包括js css images font
│   └── templates  #模板静态文件，express工程的视图模板，可由webpack打包自动生成
│       ├── about.html
│       ├── error.html
│       └── ***.html
├── webpack.config.js    #webpack配置 这个只是简单的调用框架提供的webpack功能
└──   README.md            #项目说明
```
