---
title: 记一次线上HTTP请求偶现延迟排查
date: 2020-01-27 22:00:30
tags: [DNS, TCP_NODELAY]
categories: 
- DNS
---

## 问题现场
前后端部署在客户的一个内网的物理机，这个物理机有外网 IP 和 DNS 服务，后端部署了一部分到了阿里云的 ECS 上，现在的调用链路是这样的：
```
        1             2                3                4               5
浏览器 ----> 内网机器 ----> 内网DNS服务器 -----> 内网防火墙 -----> 阿里云SLB ----> 阿里云 ECS
```

<!-- more -->

浏览器访问时，接口偶尔会出现响应超时。

## 排查
先排除外部因素，从第5环节开始检查：
- 刷新浏览器查看日志，可以看到接口超时的时候，请求并没有过来到ECS， 临时开个白名单自己电脑上调用SLB的域名，接口无响应超时，排除第5环节
- 内网机器本身也有调用ECS其他端口的代理，不过是用 SLB 的 IP 进行访问的，所以现在怀疑是DNS解析的问题
- PING 下域名发现的确有明细的卡顿延迟, 在内网机器上配置host后即解决问题

## 更细节的排查
由于内网环境原因，没有进行更细节的排查，不过参考了[简单的-http-调用为什么时延这么大](http://www.disheng.tech/blog/%E7%AE%80%E5%8D%95%E7%9A%84-http-%E8%B0%83%E7%94%A8%E4%B8%BA%E4%BB%80%E4%B9%88%E6%97%B6%E5%BB%B6%E8%BF%99%E4%B9%88%E5%A4%A7/)，路过的可以看看链接的内容，这个短文只是提供了另外的想法和解决思路，希望可以帮到。



