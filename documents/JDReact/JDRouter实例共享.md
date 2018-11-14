# JDRouter 实例共享

## 问题

&emsp;&emsp;现在项目中的路由跳转使用了 JDRouter，后来遇到一个问题是需要在路由组件外部且同级的元素中控制路由的跳转，由于 ReactNative 不同于 Web 应用，不能通过 URL 控制路由的跳转以和获取路由信息，而 ReactNative 对象传递也只是通过 context 或 props 向下级传递。

&emsp;&emsp;在使用 JDRouter 时可以在每个 Route 的组件中通过 context 获取到 router 对象，但是对于导航栏或者 tab 页这种东西虽然可以单独做一个组件然后在每个组件中使用，但是这样会造成相同组件在每次访问的时候重复渲染，而且会增加组件功能的耦合度。

&emsp;&emsp;解决思路就是将 JDRouter 对象实例共享出来， 使其能够在其他组件中  被获取到；可以通过 Redux 和 ref 属性两种方式实现

## 方法 1 - 使用 Redux 共享 router 对象

### Router 对象分析

&emsp;&emsp;要想在 Redux 中使用 Router 对象就需要把 router 对象放到 state 里，经过源码分析发现 JDRouter 中的 router 对象是 NativeHistory 类型的实例，在 Component 的 componentWillMount 方法中进行实例化，在 Router.js 中只是把 router 对象放到了 context 中，并在 getChildContext 方法里向下级传递对象。

```javascript
componentWillMount() {
    this.allowSlidePopVC = true;
    if (!this.routeStack) {
        var routeStack = {}, router = new NativeHistory();
        React.Children.forEach(this.props.children, (child, index) => {
            let { key, props: childProps } = child;
            routeStack[key] = {
                screen: childProps.component,
                sceneConfig: childProps.sceneConfig,
            }
            router.addRoute(key, childProps);
        });
        const stackNavigatorConfig = {
            headerMode: 'none',
            backAndroidListener: router.goBack.bind(router)
        };
        if (this.props.initRoute) {
            stackNavigatorConfig.initialRouteName = this.props.initRoute.key;
            stackNavigatorConfig.initialRouteParams = this.props.initRoute.routeParams;
        }
        this.routeStack = StackNavigator(routeStack, stackNavigatorConfig);
        router.setRouter(this.routeStack.router);
        this.router = router;
    }
    // this._privilegeCheck();
},
```

&emsp;&emsp;在 Router.js 中对象实例化后对 router 对象的处理还没有全部完成，只有在 componentDidMount 方法中会进行一个 setNavigator 和 setNavState 操作，设置当前路由信息对象和跳转到默认路由，setNavigator 的参数传递的是 StackNavigator 对象描述了路由的相关信息及方法，关于这个就先不多做说明了。

```javascript
componentDidMount() {
    this.router.setNavigator(this.navigator);
    this.setNavState(this.navigator.state.nav);
},
```

### 扩展 Router 对象

&emsp;&emsp;已经知道了 router 对象是如何定义的，只需要触发一个 action，并将 router 对象传递过去就行了，但最好不要修改 JDRouter 的原生代码，毕竟如果 JDRouter 有更新还要同步更新修改后的 JDRouter 实现，因此这里利用 es 语法的继承特性，创建一个自定义的 Router 对象。

```javascript
import { ReactInstance, ReactNode } from "react";

const { JDRouter, JDText } = require("@jdreact/jdreact-core-lib");
const { Router, Route } = JDRouter;

interface RouteProps {
    onNavStateChanged?: (navState: any) => {};
    onRouterReady?: (router: any) => {};
}

export default class ShareRouter extends Router<RouteProps> {
    readonly props: Readonly<{ children?: ReactNode }> & Readonly<RouteProps>;
    state: Readonly<any>;
    context: any;
    refs: {
        [key: string]: ReactInstance;
    };

    constructor(props: RouteProps) {
        super(props);

        this.state = super.state;
        this.context = super.context;
        this.refs = super.refs;

        // 不知为什么Router中自定义的方法就是不能直接override
        this.setNavState = this._setNavState.bind(this);
    }

    _setNavState(navState: any) {
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

&emsp;&emsp;这里 override 了 componentDidMount 方法，执行了一个 onRouterReady 事件并传递 router 对象，新的 Router 在组件引用时需要利用 redux 绑定 props 里的 onRouterReady 事件，将 router 对象传递给 state，在其他组件中只需要接收这个 state 里的 router 对象

## 方法 2 - 使用 ref 属性

&emsp;&emsp;将 router 放到 state 里实现共享 router 过程比较麻烦，这里还有一个简单的方法也能实现共享 router 对象，可以把  利用 React 的 ref 属性，先将 Router 对象的实例放到 Component 的 ref 属性里，之后在 Router 对象所属的容器中将 ref 中 router 对象放到 context 中传递给下级

```javascript
import React, { Component } from "react";
import PropTypes from "prop-types";
import ShareRouter from "./ShareRouter";
import Home from "../layout/Home";
import All from "../layout/All";
import Mine from "../layout/Mine";
import { View } from "react-native";
import BottomNavigation from "./BottomNavigation";
import TopNavigation from "./TopNavigation";

const Detail = require("../layout/Detail");
const { JDRouter, JDText } = require("@jdreact/jdreact-core-lib");
const { Route } = JDRouter;

export default class RouteContainer extends Component<any, any> {
    static childContextTypes = {
        router: PropTypes.object
    };

    constructor(props: any) {
        super(props);

        this.state = {};
    }

    render() {
        return (
            <View style={{ flex: 1 }}>
                <TopNavigation />
                <Router ref="router">
                    <Route key="home" component={Home} type="replace" />
                    <Route key="all" component={All} type="replace" />
                    <Route key="mine" component={Mine} type="replace" />
                    <Route key="detail" component={Detail} type="replace" />
                </Router>
                <BottomNavigation
                    router={this.refs.router}
                    route={this.state.route}
                />
            </View>
        );
    }

    getChildContext() {
        return {
            router: this.refs.router
        };
    }
}
```

&emsp;&emsp;这样做 router 实例既能在通过 context 向下传递，又能通过下级对象的 props 传递

&emsp;&emsp;但是这种方法有一个缺陷，就是 router 对象又被限制在了一个容器对象中，假如作为容器的对象本身也是嵌套在了另一个容器中，而作为上级容器也要控制路由跳转，这样还是存在 router 对象不能直接获取问题，不如 redux 共享灵活

## 存在问题

&emsp;&emsp;如果采用将 router 对象放到 state 这种方法，在调用 router 跳转方法时会出现卡顿现象
