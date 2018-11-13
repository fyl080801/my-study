# 给 Gitbook 文档中代码加上行号

&emsp;&emsp;在编写技术性文档时经常需要插入一些代码，Markdown 默认支持插入代码，方法是使用六个\`符号将代码包含起来，例如：

    ```javascript
    componentDidMount() {
        this.router.setNavigator(this.navigator);
        this.setNavState(this.navigator.state.nav);
    },
    ```

&emsp;&emsp;实际的效果就是这样的

```javascript+lineNumbers:false
componentDidMount() {
    this.router.setNavigator(this.navigator);
    this.setNavState(this.navigator.state.nav);
},
```

&emsp;&emsp;如果使用 gitbook 发布 md 文档，可以通过使用`sunlight-highlighter`插件给代码块加入行号

&emsp;&emsp;首先在文档项目中安装`sunlight-highlighter`插件，项目目录打开终端运行`npm install gitbook-plugin-sunlight-highlighter`，然后修改项目中 book.json 文件内容，在 plugins 下加入`-highlight`和`sunlight-highlighter`，并在 pluginsConfig 下添加 sunlight-highlighter 配置

```json+lineNumbers:false
{
    "title": "学习笔记",
    "plugins": [
        "splitter",
        "expandable-chapters",
        "-highlight",
        "sunlight-highlighter"
    ],
    "pluginsConfig": {
        "expandable-chapters": {},
        "sunlight-highlighter": {
            "theme": "dark",
            "lineNumbers": true
        }
    }
}
```

&emsp;&emsp;重新生成文档后刷新页面可直接看到效果

```javascript
componentDidMount() {
    this.router.setNavigator(this.navigator);
    this.setNavState(this.navigator.state.nav);
},
```

&emsp;&emsp;如果单独设置某一代码块是否启用行号，也可以通过代码块上进行单独设置实现，原 <code>\`\`\`javascript</code> 改成 <code>\`\`\`javascript+lineNumbers:false</code>
