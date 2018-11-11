# 使用 Gitbook 编写文档

## 相关说明

&emsp;&emsp;GitBook 是一个基于 npm 的命令行工具，可使用将 Markdown 格式的文件转换成 html 格式 ，制作出精美的电子书，GitBook 不是 git 的教程且跟 git 没有任何关系，关于 GitBook 更详细说明可以去看[gitbook 的官方文档](https://toolchain.gitbook.com/config.html)

&emsp;&emsp;Markdown 是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

&emsp;&emsp;github 就不多介绍了，在 github 上建立一个项目后可以设置静态内容作为项目的首页，利用 github 的这个功能结合 gitbook 文档生成功能 ，能够免费发布在线文档

## Gitbook 命令相关

&emsp;&emsp;在使用 Gitbook 时有几个重要的命令需要先说明一下

| 命令                                                  | 说明                         |
| :---------------------------------------------------- | :--------------------------- |
| `gitbook init <源地址>`                               | 初始化文档结构               |
| `gitbook serve --port <端口号> <源原地址> <目标地址>` | 启动本地服务在本地直接预览 |
| `gitbook build <源原地址> <目标地址>`                 | 转换md，构建文档             |
&emsp;&emsp;其中`init`命令执行时会根据gitbook的目录定义文件`SUMMARY.md`的内容自动生成相应的目录和文件，这个在定义目录结构时很有用

## 编辑工具

&emsp;&emsp;发布文档需要先编辑文档，这里使用 vscode 作为编辑器，主要是 vscode 支持 md 格式的文件内容格式化，可以使用格式化快捷键将文档结构标准化，还有就是可以实时预览编辑的内容。  

![alt](/images/E922D073-FACE-43A6-84AD-F065E704F90D.png)

&emsp;&emsp;另一方面 vscode 还有一个针对 Gitbook 的扩展，能够直接查看文档的目录结构并直接编辑目录结构。

## 构建项目的具体方法

&emsp;&emsp;首先现在本地 npm 环境安装 gitbook 的命令集

    npm install gitbook-cli -g

&emsp;&emsp;之后在 github 上建立一个项目，添加一个 README.md 文件，然后克隆到本地，在本地执行 `npm init` ，在 package.json 中添加 gitbook 相关的脚本，执行 `npm install`

&emsp;&emsp;package.json 里的具体定义:

```
{
    "name": "my-study",
    "version": "1.0.0",
    "description": "学习笔记",
    "main": "index.js",
    "scripts": {
        "start": "gitbook serve --port 2233 ./ ./docs",
        "renew": "gitbook init ./",
        "build": "gitbook build ./ ./docs"
    },
    "repository": {
        "type": "git",
        "url": "git+https://github.com/fyl080801/my-study.git"
    },
    "author": "",
    "license": "ISC",
    "bugs": {
        "url": "https://github.com/fyl080801/my-study/issues"
    },
    "homepage": "https://github.com/fyl080801/my-study#readme",
    "dependencies": {
        "gitbook-plugin-expandable-chapters": "^0.2.0",
        "gitbook-plugin-splitter": "0.0.8"
    }
}
```

&emsp;&emsp;其中 `gitbook-plugin-expandable-chapters` 和 `gitbook-plugin-splitter` 两个依赖项是 gitbook 插件，之后再做解释  

&emsp;&emsp;创建 `SUMMARY.md` 文件，这个是项目的目录定义文件，先定义好文档的目录结构以及文档的链接，以便在执行初始化时直接生成好对应的文件，内容如下  

```
# Summary

*   [介绍](README.md)
*   日常
    *   [使用 Gitbook 编写文档](documents/日常/使用Gitbook编写文档.md)
    *   [测试用文档](documents/日常/测试用文档.md)
*   ReactNative
    *   [使用 vscode 开发 rn](documents/ReactNative/使用vscode开发rn.md)
    *   [部署 app 到模拟器](documents/ReactNative/部署app到模拟器.md)
*   网络
    *   [如何科学上网](documents/网络/如何科学上网.md)
*   总结
    *   [2018-11-11](documents/总结/2018-11-11.md)
```

&emsp;&emsp;这就是一个 markdown 按照层次划分的无序列表，要想生成文件则需要把列表中的项写成链接格式，目录可以是无链接的，直接写成文本

&emsp;&emsp;定义完成目录后执行 `npm run renew` 实际上就是在 package.json 中定义的 `gitbook init ./` 命令

&emsp;&emsp;最后项目的目录及文件结构如下，gitbook 自动生成了 document 目录和相应的 md 文件

```
.
├── README.md
├── SUMMARY.md
├── documents
│   ├── ReactNative
│   │   ├── 使用vscode开发rn.md
│   │   └── 部署app到模拟器.md
│   ├── 总结
│   │   └── 2018-11-11.md
│   ├── 日常
│   │   ├── 使用Gitbook编写文档.md
│   │   └── 测试用文档.md
│   └── 网络
│       └── 如何科学上网.md
├── images
│   └── page-1-�\205��\203��\225�\224�.png
└── package.json
```

&emsp;&emsp;每次要追加文档内容时可先编辑 `SUMMARY.md` 的内容然后执行命令更新目录，也可安装 vscode 的 gitbook 插件 `Gitbook kit` 来管理目录及文件，具体不多作说明  

&emsp;&emsp;这里还要说明一下 package.json 文件中 `gitbook-plugin-expandable-chapters` 和 `gitbook-plugin-splitter` 这两个依赖项，分别是 gitbook 的目录折叠插件和目录宽度可拖动插件，除了在 npm 包引用里定义外，还需要在项目根目录下定义一个 book.json 文件，这个是 gitbook 相关的配置文件, 里面 plugins 属性就是要引用的插件  

```
{
    "title": "学习笔记",
    "plugins": ["splitter", "expandable-chapters"],
    "pluginsConfig": {
        "expandable-chapters": {}
    }
}
```

&emsp;&emsp;编辑完内容后可执行命令 `npm start` 启动服务在本地预览网页上的效果，也可以执行 `npm run build` 构建项目，之前在 package.json 中定义了输出目录是 docs（在 github 里只能从项目根目录或一个分支下的 docs 目录访问静态页面），目录下直接生成了一个静态网站  

![实际效果](/images/0E60DCA9-8271-4B66-99F5-0E63257FF927.png)  

&emsp;&emsp;生成好了之后只要执行 git 命令将项目同步到 github上 就可以了，接下来就是如何在 github 上显示静态内容

## 如何在 github 上设置静态网站

&emsp;&emsp;github项目中有个 Settings 选项，进入该选项  

![alt](/images/08A761FF-8A57-46EE-8F39-F3F5233F3CAE.png)

&emsp;&emsp;在 Settings 界面里有个 GitHub Pages 的地方，里面的下拉框处选择后面带 docs 那个，之前项目里生成的静态网站内容目标目录就是 docs，选好后保存一下，可以直接点击上面的生成的 url 就可以看到发不好的效果  

![alt](/images/7A595CB5-59C8-420A-80F2-02AD3D301E46.png)  

## 总结

&emsp;&emsp;没啥好说的，其实有一堆做这种在线文档的平台，自己架设的好处就是不用注册一堆账号，到时候收到一堆推广的邮件和短信，还有就是利用 github 自己动手丰衣足食  

&emsp;&emsp;其实那些做这种文档应用的平台一般都是一个管理界面+在线markdown编辑器，然后在收到用户提交的请求后更新那个 `SUMMARY.md` 和相应的 md 文件，最后再 task 一下调用 gitbook 的命令生成静态内容