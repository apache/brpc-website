---
title: "bRPC 1.1.0"
linkTitle: "bRPC 1.1.0"
weight: 3
date: 2022-04-12
description: >
  Apache bRPC 1.1.0 版本发布
---
## Apache bRPC 1.1.0 发布，支持IPV6和UDS
很高兴通知大家，Apache bRPC (孵化中) 1.1.0版本发布！

下载链接：https://brpc.apache.org/docs/downloadbrpc/

Github Release Tag：https://github.com/apache/incubator-brpc/releases/tag/1.1.0

### 1.1.0版本的主要变更：
新功能
* 支持IPV6和UDS(Unix Domain Socket) by @wwbmmm in #1560
* 支持protobuf 3.19.x by @hcoona in #1679
* 支持http协议的dump/replay by @guodongxiaren in #1503
* 支持nshead协议的dump/replay by @wwbmmm in #1486
* 支持http body为proto-text格式的request/response by @hiberabyss in #1690
* baidu_std协议支持传递client端设置的超时到server端 by @wwbmmm in #1472
* bthread创建时支持通过attr指定继承tls span by @wwbmmm in #1494
* rpc_replay支持bazel编译 by @taoxu in #1677
* Server新增 Start(PortRange, const ServerOptions*) 接口 by @serverglen in #1460
* FlapMap新增 insert(const std::pair<key_type, mapped_type>& kv) 接口 by @serverglen in #1468
* Server新增eps bvar输出 by @serverglen in #1483

Bug修复
* 修复 CheckHealth 未设置 has_request_code by @serverglen in #1502
* 修复server处理stream创建请求过程出错时发送非预期数据 by @jenrryyou in #1516
* 修复 LA selection runs too long 的出错  by @KaneVV1 in #1567
* 修复 http收到不合法请求时返回错误的response  by @jl2005 in #1620
* 修复 bvar status 编译错误 by @zwkno1 in #1625
* 优化 InputMessenger client 端重试策略 by @ehds in #1680
* 修复 work_stealing_queue_unittest 在ARM下编译错误 by @TKONIY in #1709
* 修复 LatencyRecorder qps 统计不精确 by @wwbmmm in #1708
* 修复在 gcc11 下开启 --std=c++20时编译错误 by @hiberabyss in #1719
* 修复不稳定和UT链接错误by @wwbmmm in #1711
* 修复 Thrift 下载 url 错误 by @yangzhg in #1725
* 删除 grpc ParseH2Settings 不必要的warning日志 by @yanjianglu in #1599

其它改进
* 文档改进 by @wwc7654321, @wwbmmm, @tanzhongyi003, @mahongweichina, @cdjingit, @dl239, @ehds
* 修正拼写错误 by @yangzhg, @egolearner, @PengyiPan, @Aaaaaaron, @ehds, @JiaoZiLang, @mapleFU

感谢上述所有1.1.0版本的贡献者！