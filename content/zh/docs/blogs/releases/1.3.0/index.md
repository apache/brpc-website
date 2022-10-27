---
title: "bRPC 1.3.0"
linkTitle: "bRPC 1.3.0"
weight: 1
date: 2022-10-25
description: >
  Apache bRPC 1.3.0 版本发布
---
## Apache bRPC 1.3.0发布，支持arm64平台和多维度bvar
很高兴通知大家，Apache bRPC (孵化中) 1.3.0版本发布，支持arm64平台和多维度bvar，同时Star数量也突破14k，并新增四位企业级用户：欢聚时代、Doris、BaikalDB和数美科技（nextdata）。  

bRPC官网：https://brpc.apache.org/  

下载链接：https://brpc.apache.org/docs/downloadbrpc/  

Github Release Tag：https://github.com/apache/incubator-brpc/releases/tag/1.3.0

### 1.3.0版本的主要变更：
新功能
* 支持 Linux arm64 平台的 gcc 编译 by @jamesge
* 支持 11.2.0 以上版本的 gcc 编译 by @wwbmmm in #1783
* 支持最新版本的 bazel 编译(默认v4.2.2) by @hcoona in #1657
* 支持多维度 bvar，一个功能强大的bvar扩展 by @serverglen in #1608
* 重构了 event_dispatcher 源文件 by @guodongxiaren in #1888
* 新增 Http 协议错误码重试策略 by @chenBright in #1927
* 新增 Nacos 命名服务 by @yyweii in #1922
* 允许自定义 server 端 bvar 名字前缀 by @jenrryyou in #1854
* 新增转义日志 escape_log 选项 by @jamesge

Bug修复
* 修复 FlatMap 中的一些问题 by @jamesge
* 修复 pb 中的 override 问题 by @jamesge,@wwbmmm
* 修复 ALIGNAS/ALIGNOF/BAIDU_CACHELINE_ALIGMENT  by @jamesge
* 修复了一些关于 clang 的警告并恢复 ALIGNAS/ALIGNOF 上的更改 by @jamesge
* 修复 rpc_replay 初始化 channel 失败时返回 -1 的问题 by @ehds in #1938
* 修复多维度 bvar 的编译错误 by @dabao085 in #1937
* 修复 bvar_dump_tabs 的默认值问题 by @yyweii in #1920
* 修复 butex_wait 因超时而失败的问题 by @Huixxi in #1917
* 修复 rpc_replay 无法均匀发送请求的问题 by @bumingchun in #1910
* 修复因 DumpOptions 对象导致的编译警告问题 by @ml-haha in #1905
* 修复 m1 mac 上的 test_bvar 失败问题 by @wwbmmm in #1901
* 修复 brpc_socket_unittest.cpp 中的执行缓慢问题 by @zyearn in #1898
* 修复 worker pthread 上的第一个 bthread keytable 会被删除两次的问题 by @chenBright in #1884
* 修复当前损坏的 MacOS 构建问题 by @zyearn in #1871
* 修复 ProcessHttpRequest 对 http2 的支持问题 by @dandyhuang in #1868
* 修复 get_value core 导致采样器线程启动过早的问题 by @Huixxi in #1863
* 修复 UDS 单元测试在 MacOS 上失败的问题 by @wwbmmm in #1843
* 修复由 bad growth_non_responsive http 请求导致的 coredump 问题 by @acelyc111 in #1278
* 修复使用 unicode 字符串检查 errorno 时不中止的问题 by @tobegit3hub in #1142

其它改进
* 文档改进@wwbmmm, @JackBoosY, @morningman, @serverglen, @chenBright, @guodongxiaren, @xdh0817, @KaneVV1, @tanzhongyi003, @lzfhust, @Huixxi
* 修正错别字@opheliaKyouko, @day253, @chenBright, @fansehep

感谢所有关心和为bRPC做出贡献的开发者！