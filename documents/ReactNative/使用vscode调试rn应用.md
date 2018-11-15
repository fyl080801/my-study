# 使用 vscode 调试 rn 应用

&emsp;&emsp;vscode 可以作为 rn 的调试工具，断点直接打到源码上，调试时可直接使用开发环境的编辑器跟断点和查看变量

&emsp;&emsp;先安装 vscode 的插件“React Native Tools”，之后在调试下添加相关配置项

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to packager",
            "program": "${workspaceRoot}/.vscode/launchReactNative.js",
            "type": "reactnative",
            "request": "attach",
            "sourceMaps": true,
            "outDir": "${workspaceRoot}/.vscode/.react"
        },
        {
            "name": "Debug Android",
            "program": "${workspaceRoot}/.vscode/launchReactNative.js",
            "type": "reactnative",
            "request": "launch",
            "platform": "android",
            "sourceMaps": true,
            "outDir": "${workspaceRoot}/.vscode/.react"
        },
        {
            "name": "Debug iOS",
            "program": "${workspaceRoot}/.vscode/launchReactNative.js",
            "type": "reactnative",
            "request": "launch",
            "platform": "ios",
            "sourceMaps": true,
            "outDir": "${workspaceRoot}/.vscode/.react"
        }
    ]
}
```

&emsp;&emsp;运行 `npm start`启动进程，在 debug 下选择“Attach to packager”，手机上也要开启远程调试，刷新手机上的页面之后会自动附加到调试进程

> 有时候会出现附加不上的情况，手机上多刷新几次就能成功
