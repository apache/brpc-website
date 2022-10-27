---
title: "bRPC 1.2.0"
linkTitle: "bRPC 1.2.0"
weight: 2
date: 2022-08-07
description: >
  Apache bRPC 1.2.0 版本发布
---
## Apache bRPC 1.2.0发布，支持Apple Silicon
很高兴通知大家，Apache bRPC (孵化中) 1.2.0版本发布，支持Apple Silicon。

下载链接：https://brpc.apache.org/docs/downloadbrpc/

Github Release Tag：https://github.com/apache/incubator-brpc/releases/tag/1.2.0

### 1.2.0版本的主要变更：
新功能
* 支持 Apple Silicon by @jamesge
* 增加 json选项支持数组序列化/反序列化 by wasphin in #1604
* 支持 butex_wake_all nosignal flag by @yanglimingcn in #1751
* 支持 wr/wrr 负载均衡策略设置默认权重值 by @Huixxi in #1571
* cmake 支持with_snappy编译选项 by @renzhong in #1799

Bug修复
* 增加 redis 请求为空检查 by @lzfhust in #1745
* 增加 iobuf append 返回值检查 by @wwbmmm in #1762
* 编译 protoc-gen-mcpack 增加 CXXFLAGS选项 by @jamesge
* 修改 brpc/periodic_task.h 头文件风格 by @TousakaRin in #1820
* 增加 ByteSizeLong 代替ByteSize（deprecated）函数 by @yangzhg in #1723
* 修复 C Struct编译错误 by @wolfdan666 in #1736
* 修复 自动检查https 问题 by @renzhong in #1754
* 修复 json2pb::JsonToProtoMessage() 支持解析多个json by @jamesge
* 修复 缺少limits头文件编译错误 by @GOGOYAO in #1764
* 修复 ClearAbandonedStreamsImpl死锁问题 by @zyearn in #1781
* 修复 在ClearAbandonedStreams的时候更新Windowsize问题 by @zyearn in #1786
* 修复 _dl_sym未定义问题 by @wwbmmm in #1784
* 修复 thrift协议处理机制 by @lzfhust in #1790
* 修复 编译静态库的编译错误 by @stdpain in #1797
* 修复 discovery naming service core by @serverglen in #1802 
* 优化 rpc_press 发压抖动问题 by @bumingchun #1763
* 修复 dns域名解析机器列表过长导致 channel初始化失败问题 by @chenBright in #1818

其它改进
* 文档改进 by @wwbmmm, @tanzhongyi003, @guodongxiaren, @372046933, @wasphin, @hawkxiang, @TousakaRin, @Huixxi, @serverglen
* 修正拼写错误 by @jamesge, @cyberkillor

感谢所有关心和为bRPC做出贡献的开发者！