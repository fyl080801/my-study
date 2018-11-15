# typescript 类继承原理

## 实现继承

在 typescript 中可以实现类型的继承，例如有一个基类具有如下定义

```typescript
export default class Base {
    testFunc() {
        console.log("base class");
    }
}
```

现在创建一个派生类

```typescript
import Base from "./base";

export default class aClass extends Base {
    // override 了 testFunc 方法
    testFunc() {
        super.testFunc();
        console.log("a class");
    }
}
```

在类型 Base 的派生类 aClass 中的 testFunc 方法使用 super 来调用基类的方法

使用 tsc 对 typescript 源码进行编译， 可得到编译后最终的 javascript 输出

其中基类 Base.js 中可以看出 typescript 对于类的定义实际上就是返回一个函数，类型的方法全部定义在这个函数的 prototype 中，prototype 表示该函数的原型，在类型被实例化后 prototype 对象的成员都成为实例化对象的成员

```javascript
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
var Base = /** @class */ (function() {
    // 定义一个函数 Base
    function Base() {}

    // 函数的 prototype 属性加一个成员函数
    Base.prototype.testFunc = function() {
        console.log("base");
    };

    // 返回这个函数 Base
    return Base;
})();
exports.default = Base;
//# sourceMappingURL=base.js.map
```

编译后生成的派生类 aClass.js 中首先定义了一个 \_\_extends 方法，这个方法实际上就是为了实现继承用的，接收派生类和基类两个参数，里面对基类的 prototype 指向进行了处理，将基类的 prototype 里的所有成员复制一份到派生类型中，这样会在创建派生类的实例时使得派生类具有基类所有成员（这里先不考虑基类里的私有成员），这些是基类成员的浅副本

如果派生类中要 override 基类中的成员函数，则先调用 \_\_extends 获得了基类中 prototype 的浅副本，之后在本身的 prototype 中重新定义要 override 成员函数，override 的方法里可使用 super 获取基类的引用执行基类的操作，实际上是调用了基类的 prototype 中的成员函数

```javascript
"use strict";

// 用于之后派生类型中扩展用
var __extends =
    (this && this.__extends) ||
    (function() {
        // 这是一个递归调用的过程，目的是将基类里的成员的所有属性深度复制到派生类原型（prototype）中
        // 这个方法接收两个参数，d - 派生类， b - 基类
        var extendStatics = function(d, b) {
            extendStatics =
                Object.setPrototypeOf ||
                ({ __proto__: [] } instanceof Array &&
                    function(d, b) {
                        d.__proto__ = b;
                    }) ||
                function(d, b) {
                    // 这里是复制过程 d[p] = b[p]
                    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
                };

            // 返回 extendStatics 做方法调用递归，实现遍历复制基类里所有成员
            return extendStatics(d, b);
        };
        return function(d, b) {
            extendStatics(d, b);
            function __() {
                this.constructor = d;
            }
            d.prototype =
                b === null
                    ? Object.create(b)
                    : ((__.prototype = b.prototype), new __());
        };
    })();
var __importDefault =
    (this && this.__importDefault) ||
    function(mod) {
        return mod && mod.__esModule ? mod : { default: mod };
    };
Object.defineProperty(exports, "__esModule", { value: true });
var base_1 = __importDefault(require("./base"));
var aClass = /** @class */ (function(_super) {
    // 调用 __extends 方法对 aClass 做派生处理
    // _super 是基类
    __extends(aClass, _super);
    function aClass() {
        return (_super !== null && _super.apply(this, arguments)) || this;
    }

    // 派生类里重新定义 prototype 中成员函数 testFunc 实现 override
    aClass.prototype.testFunc = function() {
        // 派生类 override 后的方法中有个 super.testFunc()
        // 其实是调用基类 prototype 的成员函数，并将当前派生类的 this 传过去
        _super.prototype.testFunc.call(this);
        console.log("a class");
    };
    return aClass;
})(base_1.default);
exports.default = aClass;
//# sourceMappingURL=aClass.js.map
```

## 之前在 ReactNative 的问题

之前在 ReactNative 中如果是使用 React.createClass 创建的组件类型，会在 createClass 传递一个对象，直接在对象里定义了一个成员方法，虽然另一个基于 typescript 写的组件可以继承这个类， 但是由于成员方法不是写在 prototype 中的，在 typescript 编译后执行时 prototype 里获取不到这个成员，在被调用时原方法只存在于基类的实例中，因此派生类中 override 的方法不会被调用，而原生的方法例如 componentDidMount、render 等是在 React.createClass 过程中做了处理

```javascript
// Router 是通过 React.createClass 定义的
React.createClass({
    // 里面有一个方法
    setNavState(navState: any) {
        // ...
    }
});
```

```typescript
// 用 typescript 实现一个派生类
export default class ShareRouter extends Router<RouteProps> {
    constructor(props: RouteProps) {
        super(props);
    }

    // 这里直接 override，在被调用时不会执行这个 override 后的方法的
    // 因为经过 typescript 编译后基类 prototype 不存在这个成员，基类中这个法在派生类中还是指向了原方法
    setNavState(navState: any) {
        super.setNavState(navState);
        (this.props.onNavStateChanged || ((navState: any) => {}))(navState);
    }

    componentDidMount() {
        super.componentDidMount();
        (this.props.onRouterReady || ((router: any) => {}))(this.router);
    }

    render() {
        return super.render();
    }

    setState(
        state:
            | ((
                  prevState: Readonly<any>,
                  props: Readonly<RouteProps>
              ) => Pick<any, any> | RouteProps | null)
            | (Pick<any, any> | any | null),
        callback?: () => void
    ) {
        super.setState(state, callback);
    }

    forceUpdate(callBack?: () => void) {
        super.forceUpdate(callBack);
    }
}
```

因此，如果想 override 这样的方法只能在派生类中手动完成 typescript 编译结果的指向实现

```typescript
export default class ShareRouter extends Router<RouteProps> {
    constructor(props: RouteProps) {
        super(props);

        // 需要把 this（本类型实例）中 setNavState 手动指向派生类中定义的方法
        this.setNavState = this._setNavState.bind(this);
    }

    // 定义一个 override 后的方法
    _setNavState(navState: any) {
        super.setNavState(navState);
        (this.props.onNavStateChanged || ((navState: any) => {}))(navState);
    }

    // 这种 react 原生的方法会在 React.createClass 过程中完成指向的这个过程，因此不需做处理
    componentDidMount() {
        super.componentDidMount();
        (this.props.onRouterReady || ((router: any) => {}))(this.router);
    }

    render() {
        return super.render();
    }

    setState(
        state:
            | ((
                  prevState: Readonly<any>,
                  props: Readonly<RouteProps>
              ) => Pick<any, any> | RouteProps | null)
            | (Pick<any, any> | any | null),
        callback?: () => void
    ) {
        super.setState(state, callback);
    }

    forceUpdate(callBack?: () => void) {
        super.forceUpdate(callBack);
    }
}
```
