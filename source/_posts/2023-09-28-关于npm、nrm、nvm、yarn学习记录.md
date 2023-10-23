---
title: 关于npm、nrm、nvm、yarn学习记录
date: 2023-09-28 10:50:04
tags: npm、nrm、nvm、yarn
categories: [前端技术,nodejs]
---

***


# 一、npm
###### 1、什么是npm？
npm 是 Node.js 的包管理器，用于安装、管理、卸载 JavaScript 模块。npm 提供了一个包管理器，使得开发者可以方便使用第三方模块，同时也可以将自己编写的模块发布到 npm 上供其他人使用。（npm相当于后端的maven，帮助我们下载依赖）
**注意：**
	npm不用安装，只要安装了nodejs就有了。
###### 2、官网地址
[https://www.npmjs.com/](https://www.npmjs.com/)
###### 3、中文官网地址
[https://www.npmjs.cn/](https://www.npmjs.cn/)
###### 4、使用方法
（1）查看npm版本
``` cmd
npm -v
```
（2）初始化一个npm项目
说明：在目录下运行下面的命名，目录中就会 多了一个文件 package.json（相当于pom.xml）
``` cmd
// -y:直接生成 package.json
npm init -y
```
（3）下载依赖包
``` cmd
// 可以简写为：npm i 【包名】
npm install 【包名】
```
**说明：**
npm install xxx --save --global（简写：-g）（简写：-S大写）-dev
- xxx 表示某个依赖名字
- --save 表示将这个依赖保存进 package.json 依赖标签里（dependencies）；
- -dev 表示将这个依赖保存进 package.json 开发依赖标签里（devDependencies）；
- --global 表示全局安装。
**注：**npm5及更高版本install 和install -S效果相同，如缺省默认为--save  

（4）同时下载多个依赖包
``` cmd
// 包名之间用空格隔开
npm i 【包名1】 【包名2】 【包名3】...
```
（5）配置npm下载镜像
``` cmd
npm config set registry http://registry.npm.taobao.org
```
（6）卸载某个依赖包
``` cmd
// 可以简写为：npm i 【包名】
npm uninstall 【包名】
```
（7）npm启动项目命令
``` cmd
// 具体查看package.json中是不是serve
npm run serve
```
（8）npm构建项目命令
``` cmd
npm run build
```
（7）查看全局按照位置
``` cmd
npm config get prefix
```
（8）如何找依赖包
	可以去上面第2点中的官网查找需要的依赖包名。
	
<br/>

***

<br/>


# 二、yarn
###### 1、什么yarn？
和npm是一样的（yarn需要使用npm安装，npm是npdejs自带的）。
###### 2、官网
（1）地址1：[https://yarnpkg.com/](https://yarnpkg.com/)
（2）地址2：[https://classic.yarnpkg.com/en/docs/install/#windows-stable](https://classic.yarnpkg.com/en/docs/install/#windows-stable)
###### 3、中文官网
（1）地址1：[https://yarn.nodejs.cn/](https://yarn.nodejs.cn/)
（2）地址2：[https://www.yarnpkg.cn/](https://www.yarnpkg.cn/)
###### 4、使用方法
（1）安装方法
``` cmd
npm install --global yarn
```
（2）查看版本
``` cmd
yarn --version
```
（3）添加依赖包
**说明：** 下载不了就加  --ignore-engines
``` cmd
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]

// 分别添加到 devDependencies、peerDependencies、optionalDependencies
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional
```
（4）升级依赖
``` cmd
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```
（5）删除依赖
``` cmd
yarn remove [package]
```

<br/>

***

<br/>


# 三、nrm
###### 1、什么是nrm？为什么要使用nrm？
我们在使用npm下载依赖的时候，如果不设置国内镜像，下载得就很慢，所以我们需要手动去设置镜像，有可能有些依赖还不能使用国内镜像下载，这个时候还得去设置镜像，虽然有cnpm，但是还是太麻烦了。而nrm就是帮助我们解决了这个麻烦事，提供了一些最常用的npm包镜像地址，能够让我们快速的切换镜像地址。
###### 2、使用方法
（1）安装方法
``` cmd
// 全局安装nrm包
npm i nrm -g
```
（2）查看当前所有可用的镜像源地址以及当前所使用的镜像源地址
``` cmd
nrm ls
```
（3）切换不同的镜像源地址
``` cmd
// 切换成npm镜像
nrm use npm

// 切换成taobao镜像
urm use taobao
```

<br/>

***

<br/>

# 四、nvm

###### 1、什么是nvm？为什么要使用nvm？
我们在使用nodejs的时候，一台电脑只能装一个版本的nodejs，如果我们想要更换nodejs版本，就需要我们卸载已有的版本，重新下载我们需要的版本，很麻烦，而nvm帮我们解决了这个麻烦事。
nvm是一个 node版本管理工具，拥有它可以轻松的让我们在一台电脑上随时切换node版本。
###### 2、官网地址
[https://nvm.uihtm.com/](https://nvm.uihtm.com/)
###### 3、使用方法
（1）设置镜像
	在 nvm 的安装路径下，找到 settings.txt，设置node_mirro与npm_mirror为国内镜像地址。
	将下面两行复制粘贴进settings.txt中。
``` txt
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/

// 或者
nvm npm_mirror https://npmmirror.com/mirrors/npm/
nvm node_mirror https://npmmirror.com/mirrors/node/
```
（2）显示nodejs可下载版本的部分列表
``` cmd
nvm list available
```
也可以打开链接查看可以node版本：https://registry.npmmirror.com/binary.html?path=node/
（3）安装最新版本 ( 安装时可以在上面看到 node.js 、 npm 相应的版本号 ，不建议安装最新版本)
``` cmd
nvm install latest
```
（4）安装指定的版本的nodejs
``` cmd
// 例如：nvm install 8.12.0
nvm install 【版本号】
```
（5）查看已安装版本
**说明：** 当前版本号前面没有 * ， 此时还没有使用任何一个版本，这时使用 node.js 时会报错。 

``` cmd
// 简写：nvm ls
nvm list
```
（6）切换node版本
**说明：**  这时会发现在启用的 node 版本前面有 * 标记，这时就可以使用 node.js。
``` cmd
nvm use 【版本号】
```

<br/>

***

<br/>

# 五、其它问题
##### 1、npx、npm、cnpm、pnpm区别
[https://zhuanlan.zhihu.com/p/494076214](https://zhuanlan.zhihu.com/p/494076214)
##### 2、npm、nrm、nvm的安装和使用
[https://www.jianshu.com/p/95d5228ac73e](https://www.jianshu.com/p/95d5228ac73e)