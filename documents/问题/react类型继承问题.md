# react 类型继承问题

## 情况 1 - 使用 React.createClass 创建的类用 typescript 写法实现继承

现用 React.createClass 创建一个类，里面有一个方法 testFunc

```jsx
"use strict";

const React = require("react");

module.exports = React.createClass({
    render: function() {
        return this.testFunc();
    },

    testFunc: function() {
        return <div>js 基类</div>;
    }
});
```

现创建一个派生类，里面 override 方法 testFunc

```tsx
import * as React from "react";

const JsBase = require("./JsBase");

export class JsClass extends JsBase {
    testFunc() {
        return (
            <div>
                {super.testFunc()}
                <div>js 派生</div>
            </div>
        );
    }
}
```

输出结果

```
js 基类
```

## 情况 2 - 使用 typescript 写法创建的类用 typescript 写法实现继承

实现一个类。里面有一个 testFunc 方法

```tsx
import { Component } from "react";
import * as React from "react";

export class TsBase extends Component {
    render() {
        return this.testFunc();
    }

    testFunc() {
        return <div>ts 基类</div>;
    }
}
```

现创建一个派生类，里面 override 方法 testFunc

```tsx
export class TsClass extends TsBase {
    testFunc() {
        return (
            <div>
                {super.testFunc()}
                <div>ts 派生</div>
            </div>
        );
    }
}
```

输出结果

```
ts 基类
ts 派生
```

## 问题

该程序已经集成了 tsc，不用管 ts 编译

现有问题如下

> 为什么使用 React.createClass 创建的类派生后不走 override 方法 testFunc  
> 为什么使用 class 定义的类派生后可以走 override 方法

获取项目

```
git clone https://github.com/fyl080801/react-inherit-test.git
```
