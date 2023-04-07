---
title: "bRPC 1.4.0"
linkTitle: "bRPC 1.4.0"
weight: 6
date: 2023-02-07
description: >
  Apache bRPC 1.4.0 版本发布
---
## Apache bRPC 1.4.0发布，支持RDMA
很高兴通知大家，Apache bRPC (孵化中) 1.4.0版本发布，支持RDMA。

bRPC官网：https://brpc.apache.org/  

下载链接：https://brpc.apache.org/downloadbrpc/  

Github Release Tag：https://github.com/apache/brpc/releases/tag/1.4.0

### 1.4.0版本的主要变更：
新功能
* 支持 RDMA by @Tuvie in #1836, #1967, #2005, #2036
* MVariable 支持导出到 Prometheus by @ldak4747 in #1964
* 支持超时限流 by @yanglimingcn in #2027
* 支持限制同一连接上消费队列大小 by @chenbay in #1958
* 支持出错时自定义 HTTP body by @jamesge
* 支持禁用采样线程 by @leaf-potato in #1990
* MVariable 添加 delete_stats 和 has_stats 接口 by @serverglen in #2041
* 优化并发 channel，支持配置各 channel 分片总数 by @cdjingit in #2057
  
Bug修复
* 修正使用 LLD 链接器时符号重复导致的链接错误 by @adonis0147 in #1936
* 修正 aarch64 下使用 clang 编译后运行时 "sched_to itself" 的问题 by @adonis0147 in #1950
* 修正解析 redis 消息时数据不足导致已解析消息被释放的问题 by @dorothy00dd2 in #1959
* 修正非 http (s) 协议误设置 host 的问题 by @thorneliu in #1973
* 修正 MacOS 下编译警告: bool literal returned from 'main' -Wmain by @zyearn in #2020
* 修正未清除先前错误信息导致的获取 ssl 错误码错误的问题 by @yyweii in #2019
* 修正 domain buffer 长度比标准小的问题 by @wayslog in #1965
* 修正 demangle core 问题 by @wwbmmm in #2037

功能增强
* 支持使用 bazel 编译 rdma_performance by @372046933 in #1984
* 支持使用 bazel 作为第三方模块被集成 by @fansehep in #1996
* 尝试加载多个可能的 libibverbs.so 路径以成功启用 RDMA by @372046933 in #1985
* 归还 socket 时更新写入时间；调整 -idle_timeout_second 默认值为 30. by @jamesge
* 当 IOBuf::append_user_data when size == 0 时尽早返回，减少不必要的处理逻辑 by @372046933 in #2009
* 支持设置自定义 json parser/writer by @old-bear in #2026
* 统一序列化 / 反序列化行为，仅支持反序列化根对象数组为单个 repeated 成员的
* protobuf 对象 by @chenBright in #2035
* FlatMap 支持 unique_ptr 类型作为 value by @jamesge

其他
* 调整警告消息，提高可读性 by @leaf-potato in #1989
* 将 CI 迁移到 GitHub workflow by @guodongxiaren and @zyearn in #1899, #2008, #2015, #2018, #2023 and #2030
* 减少单元测试输出日志 by @wwbmmm
* 支持 RHEL 9 系列发行版下 RPM 打包 by @wasphin in #1955
* 避免 std::string 拷贝 @ml-haha in #1969
* 头文件中直接包含依赖 by @372046933 in #1993
* 更新 iobuf.cpp 中警告消息，提高可读性 by @wwbmmm
* 删除不必要的分号 by @guodongxiaren in #2004
* profile graph 页面添加描述信息，以方便理解 by @hongliuliao in #2007
* 删除示例下未使用的 logoff_ms 选项 by @leaf-potato in #2064
* 尝试修复 "libbrpc.so: undefined symbol: pthread_mutex_lock" 问题 by @co0l1ce in #2049 and #2076
* 保持 bthread TaskGroup abi 兼容 by @wwbmmm in #2047
* 其他文档类改进 by @fansehep, @cuishuang, @tanzhongyi003, @lorinlee, @Huixxi, @steven-66, @serverglen, @wwbmmm, @wasphin, @Tuvie, @0xflotus, @thinh2, @leaf-potato, @TousakaRin, @cdjingit, @chenBright, @freemandealer and @yanglimingcn

感谢所有关心和为bRPC做出贡献的开发者！