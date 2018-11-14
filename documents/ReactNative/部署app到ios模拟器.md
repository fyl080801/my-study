# 部署 app 到 ios 模拟器

## 启动模拟器

&emsp;&emsp;启动模拟器使用`xcrun instruments`命令，使用`xcrun instruments -help`可查看该命令的帮助，其中`xcrun instruments -w <设备名称>`是启动模拟器

## 设置开发目录

&emsp;&emsp;在终端执行如下命令

```bash+lineNumbers:false
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer/
```

## 部署 app 到模拟器

&emsp;&emsp;在终端执行如下命令

```bash+lineNumbers:false
xcrun simctl install booted <app的路径>
```
