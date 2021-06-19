---
layout: default
title: 日志系统
parent: 自动驾驶系统
---

# 日志系统

## 需求
* 支持不同级别的log（TRACE、DEBUG、INFO、WARN、ERROR、FATAL）
* 支持写入文件及屏幕输出（标准输出/标准错误）
* 支持根据启动时的环境变量控制log行为（开关、级别、是否输出至标准输出/标准错误、是否输出至文件、log文件路径）
* 支持每n个同类log只输出一次
* 支持每n秒内同类log只输出一次
* 支持热切换log级别
* 支持异步log
* 支持编译期根据编译选项开关log

## 开源实现
比较经典的开源log实现例如glog、spdlog。目前来看，这两个log都不能完整支持以上需求，比如：glog不支持异步log，这两种log都不支持级别热切换。此外，spdlog的api为print式的api，而glog的api为stream式的api，后者更符合C++使用习惯。

## 实现技巧
实现上，基本的日志实现需要一个单例+写文件的封装，并通过一些内置的宏获取行号和文件名，比如内置的`__LINE__`、`__FUNCTION__`、`__FILE__`和如下面定义的`__FILENAME__`。
```cpp
#define __FILENAME__ \
    (strrchr(__FILE__, '/') ? (strrchr(__FILE__, '/') + 1) : __FILE__)
```

为了减少io操作的次数，可以设计一级缓冲区。

为了提高性能，可以设计一个工作线程用来从缓冲区中向文件写入。

## 参考
1. [如何编写高性能日志](https://mp.weixin.qq.com/s?__biz=MzU2MTkwMTE4Nw==&mid=2247486888&amp;idx=1&amp;sn=cd321d522175d99033aefade3d29c1f3&source=41#wechat_redirect)