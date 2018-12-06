# nodejs 简单 http 转发

## 说明

花了 10 分钟研究了一下 nodejs 的 http 相关知识，现实现一个简单的 http 请求的转发，先暂时只考虑 get 的情况

## 服务说明

转发环境需要三个 node 服务，一个用于接收请求并转发，另外两个用于最终处理请求，返回结果  
基本思路是使用 express 搭建服务，接收客户端的请求，接到请求后，使用 http 提供的 request 方法进行转发，在接到回复后将转发服务的 response 数据写回到原请求的 response 中

## 程序说明

express 是一个简易 http 服务，用于启动监听指定 http 端口的请求

http 用于实现 http 请求，实现转发

在入口服务接受到请求后的回调中会传递两个参数，request 和 response

其中 request 是请求对象，里面包含了原始请求的各种参数，实际的转发就需要把这里面的重要数据进行转发，例如授权信息或提交的信息等内容，这里先不考虑这些复杂情况

另一个对象 response 代表返回的对象，用于将请求结果写到输出里最终返回给客户端

这里简单实现了一个负载均衡的情况，建立两个服务用于轮流处理请求

## 入口服务实现

```javascript
const express = require("express");
const http = require("http");

const app = express();

let requestIndex = -1;

// 服务列表
const hosts = ["localhost:3331", "localhost:3332"];

// 负载均衡处理
const getHost = () => {
  requestIndex++;
  if (requestIndex >= hosts.length) {
    requestIndex = 0;
  }
  return hosts[requestIndex].split(":");
};

// 返回处理转发的回调函数
// 这里用柯里化法实现分步传参，接收原始请求的参数
const replyCallback = orgResponse => {
  return subResponse => {
    var result = "";

    // 返回的结果在处理数据时将数据取出来
    subResponse.on("data", body => {
      result += body;
    });

    // 结束转发调用原始 response 对象将请求结果写回去
    subResponse.on("end", () => {
      orgResponse.send(result);
    });
  };
};

// 接收一个请求
app.get("/", (orgRequest, orgResponse) => {
  var currentHost = getHost();

  // 创建一个转发对象
  let subRequest = http.request(
    {
      hostname: currentHost[0],
      port: currentHost[1],
      path: orgRequest.pathname,
      method: orgRequest.method
    },
    replyCallback(orgResponse)
  );

  // 执行转发
  subRequest.end();
});

app.listen("2233", () => console.log("入口服务启动, 端口：2233"));
```

## 接收服务实现

```javascript
const express = require("express");
const http = require("http");

const app = express();

app.get("/", (request, response) => {
  console.log("server 接到请求");
  response.send("server<服务编号>:" + "aaaaa");
});

app.listen("<服务端口号>", () => console.log("host<服务编号>:start"));
```

## 总结

nodejs 是一个基于 chrome v8 引擎的 javascript 运行环境，具有高性能的 io 处理能力，适合做 io 密集型应用，将来可以考虑更多的应用情景支持多种协议，实现类似 nginx 中的服务转发中间件功能
