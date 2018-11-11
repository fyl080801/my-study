# 使用 Gitbook 编写文档

## 相关说明

&emsp;&emsp;GitBook 是一个基于 npm 的命令行工具，可使用将 Markdown 格式的文件转换成 html 格式 ，制作出精美的电子书，GitBook 不是 git 的教程且跟 git 没有任何关系，关于 GitBook 更详细说明可以去看[gitbook 的官方文档](https://toolchain.gitbook.com/config.html)

&emsp;&emsp;Markdown 是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

&emsp;&emsp;githubj 就不多介绍了，在建立一个项目后可以设置项目中一个分支作为项目首页，利用 github 的发布首页功能结合 gitbook 文档生成功能 ，能够免费发布在线文档

## 编辑工具

&emsp;&emsp;发布文档需要先编辑文档，这里使用 vscode 作为编辑器，主要是 vscode 支持 md 格式的文件内容格式化，可以将文档结构标准化，还有就是可以实时预览编辑的内容。  
&emsp;&emsp;另一方面 vscode 还有一个针对 Gitbook 的扩展，能够直接查看文档的目录结构并直接编辑目录结构。

## 具体方法

&emsp;&emsp;首先现在本地 npm 环境安装 gitbook 的命令集

    npm install gitbook-cli -g

&emsp;&emsp;之后在 github 上建立一个项目，添加一个 README.md 文件，然后克隆到本地，在本地执行`npm init`，在 package.json 中添加 gitbook 相关的脚本

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

sss
