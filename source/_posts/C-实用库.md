---
title: C++ 实用库
date: 2019-05-13 10:11:07
tags: [C++]
---

##### 1.  [spdlog](https://github.com/gabime/spdlog)

非常好用的 C++11 日志库，支持 ***fmt*** 格式化。

* Header only
* 支持跨平台， Linux, Mac (clang 3.5+), Windows (msvc 2013+, cygwin), Android
* 支持异步 IO
* 支持自定义格式化
* 支持多线程
* 支持二进制文件格式
* 支持  Rotating, Daily, console, syslog, Windows debugger 以及自定义扩展

##### 2.  [nlohmann/json](https://github.com/nlohmann/json)

这不是一个追求性能的 json 库， 但我觉得这是用起来最方便的一个。
它可以让你像在 Python 中一样方便直观的调用，组建  json。

![](https://raw.githubusercontent.com/nlohmann/json/master/doc/json.gif)

##### 3.  [fmt](https://github.com/fmtlib/fmt)

快速且安全的格式化库，可替代 标准库 **(s)printf** 和 **iostreams**

* 相比标准库更快的速度 参见 [Benchmark](https://github.com/fmtlib/fmt#speed-tests)
* 支持自定义类型的格格化
* 类**Python** ***str.format*** 的语法风格
* 类型安全及自动内存管理，编译时错误报告
* 支持宽字符