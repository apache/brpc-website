---
title: "bRPC 1.5.0"
linkTitle: "bRPC 1.5.0"
weight: 5
date: 2023-05-07
description: >
  Apache bRPC 1.5.0 版本发布
---
## Apache bRPC 1.5.0发布
### 新功能
- DNS解析支持IPv6 by @jsl422 in #2139
- 支持为定时器采样线程和工作线程命名 by @ehds in #2136
- 支持为不同的方法配置不同的TimeoutConcurrentConf配置 @yanglimingcn in #2112
- 新增bvar is_hidden by @serverglen in #2205
- 在Status内置服务中新增服务器并发数 by @chenBright in #2097
- 新增“平均延迟”Prometheus指标 by @Huixxi in #2024

### Bug修复
- 修复在RDMA示例中常量未使用问题 by @goldenbean in #2187
- 修复域名服务中主机名缓冲区长度问题 by @ehds in #2179
- 修复Socket内存泄漏问题 by @chenBright #2169
- 修复当名字服务启动失败时无法结束等待问题 by @chenBright #2162
- 修复libprotoc路径错误导致ci失败问题 by @guodongxiaren in #2132
- 修复周期名字服务退出问题 by @chenBright in #2123

### 功能增强
- 移除wordexp by @wwbmmm in #2218
- 针对仅修改markdown文档的变更跳过不必要的工作流检查 by @kiminno in #2175
- 优化当nbucket为0时拒绝初始化FlatMap by @jamesge
- 优化一些违反C++ ODR规则的代码 by @lrita in #2161
- FlatMap和FlatSet支持自定义 allocator 内存分配器 by @old-bear in #2149
- 添加BasicStringPiece::const_pointer类型 by @lrita in #2141
- PtrContainer增加运算符重载 by @chenBright in #2107
- 优化确保至少能收到一个请求用以更新average latency by @yanglimingcn in #2106
- 优化使用rdma时的cpu开销 by @Tuvie in #2100

### 其他
- 使用env查找bash by @wasphin
- 改进/添加文档 by @haihuju, @tanzhongyi003, @wwbmmm, @wasphin, @maheshrjl, @chenBright, @NIGHTFIGHTING, @Huixxi, @zuyu, @kiminno, @wy1433, @20083017, @Thunderbrook

感谢1.5.0版本的所有贡献者！