

 
  <h1 id="create-project" class="page-header">准备</h1>


## 需要安装的软件

首先安装nodejs
然后全局安装nodemon webpack:
```
npm install nodemon webpack -g
```
## 准备工作环境
为了加快npm package下载速度，公司内部安装了npm 包服务器，按照如下方式设置即可使用。
修改npmrc文件，
windows系统： C:\Program Files\nodejs\node_modules\npm\npmrc 
macos: ~/.npmrc
添加如下行：
```
registry = http://172.16.3.105:8081/repository/npm-group/
init.author.name = USERNAME
init.author.email = USERNAME@my118.com
init.author.url = http://www.my118.com
# an email is required to publish npm packages
email=USERNAME@my118.com
always-auth=true
_auth=YWRtaW46YWRtaW4xMjM=

```

npm config set registry   npm info underscore

npm config set registry http://172.16.3.105:8081/repository/npm-group/  npm info underscore

## 推荐使用的前端编辑器

首推使用微软的前端开发神器 VSCode
[微软官方安装地址](https://code.visualstudio.com) 

其次推荐WebStorm，这个是商用软件，下载地址自己找。

 