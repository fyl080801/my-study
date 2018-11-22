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

> 1. 为什么使用 React.createClass 创建的类派生后不走 override 方法 testFunc
> 2. 为什么使用 class 定义的类派生后可以走 override 方法

> -   我知道这个跟用不用 ts 写没关系，但就是想知道为什么 React.createClass 的不行，还有 js 派生的原理，是不是像[之前分析的那样](https://fyl080801.github.io/my-study/documents/%E6%97%A5%E5%B8%B8/typescript%E7%B1%BB%E7%BB%A7%E6%89%BF%E5%8E%9F%E7%90%86.html)，继承是从 prototype 来的，React.createClass 创建完后 prototype 没有 testFunc

获取项目

```
git clone https://github.com/fyl080801/react-inherit-test.git
```
