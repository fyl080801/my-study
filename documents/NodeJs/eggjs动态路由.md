# eggjs 动态路由

## 说明

### 动机

eggjs 中定义路由需要才 controller 中加 js 文件，之后再 router 中引用

存在三个问题

- 每加一个 controller 就要在 router 里手动引用

```javascript
'use strict'

module.exports = app => {
  const { router, controller } = app
  router.get('/', controller.home.index)
}
```

- 每个 router 都需要定义 http 请求方法、请求路径和使用哪个 controller

```javascript
'use strict'

module.exports = app => {
  const { router, controller } = app
  router.get('/', controller.home.index)
  router.post('/dopost', controller.home.dopost)
  router.put('/doput', controller.home.doput)
  router.delete('/dodelete/:id', controller.home.dodelete)
}
```

- 如果多个人同时增加 controller 可能要同时修改 router 文件

### 方案

实现一个 `controller` 方法，返回一个路由生成器，对象包含 `get` `post` `put` `patch` `delete` 方法，每个方法接收两个参数，分别是最后一级路径的名称和路由的实现方法，可链式调用生成一个路由的原型对象

实现一个 ` resolveRouters ` 用于在 `router` 中调用，构建路由

## 实现

### 自动路由实现

```javascript
'use strict'

const pathUtl = require('path')
const fs = require('fs')

const factory = () => {
  var stores = []

  var func = function() {
    return (app, path) => {
      var { router } = app

      stores.forEach(item => {
        var uripath =
          '/' +
          (Array.isArray(path) ? path : [path])
            .concat([item.name])
            .filter(val => {
              return val !== ''
            })
            .join('/')

        // 可否改进一下，将一个文件中定义的路由的 this 都指向一个对象
        // 通过调用 controller 方法时传一个初值
        router[item.method](uripath, new (controllerClass(item.action))().index)
      })
    }
  }

  func.get = (name, fn) => {
    stores.push({ name: name, method: 'get', action: fn })
    return func
  }

  func.post = (name, fn) => {
    stores.push({ name: name, method: 'post', action: fn })
    return func
  }

  // delete 方法跟原生方法有冲突所以名字定义成 drop
  func.drop = (name, fn) => {
    stores.push({ name: name, method: 'delete', action: fn })
    return func
  }

  func.put = (name, fn) => {
    stores.push({ name: name, method: 'put', action: fn })
    return func
  }

  func.patch = (name, fn) => {
    stores.push({ name: name, method: 'patch', action: fn })
    return func
  }

  return func
}

/**
 * 控制器实例化类
 * @param {*} fn 路由方法
 */
const controllerClass = fn => {
  var func = function() {}

  func.prototype.index = fn

  return func
}

/**
 * 控制器工厂方法
 * @param {*} name
 */
const controller = name => {
  return factory(name)
}

/**
 * 路由生成器方法
 * @param {*} app
 * @param {*} base
 * @param {*} currentpath
 */
const resolveRouters = (app, base, currentpath) => {
  fs.readdirSync(pathUtl.join(base, currentpath)).forEach(dir => {
    var nextpath = pathUtl.join(base, currentpath, dir)

    var stat = fs.statSync(nextpath)

    if (stat.isFile()) {
      require(nextpath)()(
        app,
        nextpath				// 可以不这样用，直接从 stat 把文件名取出来
          .replace(base, '')
          .split('/')
          .filter(val => {
            return val !== ''
          })
          .map(val => {
            return val.replace('.js', '')
          })
      )
    }

    if (stat.isDirectory()) {
      resolveRouters(app, base, nextpath.replace(base, ''))
    }
  })
}

module.exports = {
  resolveRouters,
  controller
}
```

### router 中调用

在 `router` 文件中引用脚本文件，直接调用 `resolveRouters` 方法，传入 `app` 对象、根目录、路由目录

会自动生成基于根目录开始的请求

```javascript
'use strict'

module.exports = app => {
  require('./router/autoRouter').resolveRouters(app, __dirname, '/api')
}
```

### controller 定义

单独建立一个文件作为 `controller` 并引用自动路由脚本文件，先调用 `controller` 方法得到一个路由生成器，链式调用 `get` `post` `put` `patch` `delete` 方法构建路由的定义

```javascript
'use strict'

module.exports = require('../router/autoRouter')
  .controller()
  .get('index/:id', ctx => {
    ctx.response.body = 'get is called' + ctx.params.id
  })
  .post('dopost', ctx => {
    ctx.response.body = 'post is called'
  })
  .drop('delete', ctx => {
    ctx.response.body = 'delete is called'
  })
```

