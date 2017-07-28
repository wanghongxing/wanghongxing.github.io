
   <h1 id="create-page" class="page-header">新建页面</h1>


## 新建单独页面

- 负责页面HTML文件
把 src/views/index.html 复制一份，改成你想要的名字（比如：foo.html），然后修改里面的内容。
这时候在浏览器中输入 http://localhost:8080/foo ，你就能看到你新建的页面。（目前还需要重新启动运行环境，也就是执行**终止任务** **运行任务**）


- 添加简单的脚本和样式
直接在html文件

```
头部添加
<link rel="stylesheet"  
尾部添加<script>
```

就行

- 添加传统模式的脚本和样式
新建独立的css js  文件，分别保存在src/styles/css/ src/scripts/js/ 子目录下
然后在 html文件头部添加

```
<link rel="stylesheet" href="styles/css/*.js">
```

尾部添加

```
<script src="scripts/js/*.js"></script> 
```

就行.
注意：此模式下js css目录下的文件会按照目录结构复制，每个页面可以添加不受限制的js css

- 添加模块化的脚本和样式
所谓模块化就是你不用在 html 文件中添加script stylesheet标签，框架构建时候自动帮你维护依赖的资源并自动插入script stylesheet标签。




 