---
title: "bRPC 1.0.0"
linkTitle: "bRPC 1.0.0"
weight: 4
date: 2021-08-12
description: >
  Apache bRPC 1.0.0 Release Version.
---
## What's Changed
New features:
* fix grpc ut by @zyearn in #1036
* replace_include_with_find_package by @zyearn in #1032
* ignore ELIMIT for circuit breaker by @TousakaRin in #1005
* limit minimum value of max_concurrency for auto_cl by @TousakaRin in #1003
* Add missing '`' by @lingbin in #1053
* fix heap overflow in simple_data_pool by @yockie in #1056
* fix redis args by @liumh8 in #1128
* timer_thread: remove redundant code by @lorinlee in #1137
* Update nshead_service.md by @pklim101 in #1151
* fix_share_tls_block by @zyearn in #1156
* (FIX COMMENT)butil: remove BasicStringPiece in comments by @mapleFU in #1172
* fix h2_req check failed when retry after ELIMIT error by @heyuyi0906 in #1161
* fix brpc.baidu.com not accessible by @zyearn in #1189
* add .asf.yaml to set github about info by @tanzhongyi003 in #1222
* update .asf.yaml to change RPC into lowercase by @tanzhongyi003 in #1224
* add option to allow access methods from default url by @wasphin in #1214
* For https naming service, use 443 as default port by @TousakaRin in #1139
* update StreamOptions by @cstarc in #1247
* Fix build break for clang 10.0.1 by @thanksunix in #1261
* coding style fix in brpc/trackme.cpp by @gydong in #1270
* Compatibility improvement of protobuf json format and spring http spec by @chenzhangyi in #1292
* Fix bazel build in macos by @chenzhangyi in #1298
* make butil::BasicStringPiece support string split functions-family by @lrita in #1295
* make butil::ScopedVector support std::initializer_list by @lrita in #1291
* Update docs by @serverglen in #1258
* Add rpm packaging spec by @wasphin in #1290
* Fix the compile error when using GCC compiler with asan enabled on Linux platform. by @warriorpaw in #1289
* Fix registering multiple addresses to discovery by @wasphin in #1320
* Fix rpm spec by @wasphin in #1324
* Remove FlameGraph dependency by @zyearn in #1345
* Fix build failure on macOS with FlameGraph by @gogdizzy in #1411
* status文档中的代码示例写错了，需要实现的函数名应为Describe by @guodongxiaren in #1413
* Update bvar.md by @codroc in #1414
* 修改la策略中的异常日志的级别，避免在使用glog的情况下core dump by @guodongxiaren in #1415
* Change chinese quotation marks in bvar_c++.md sample code by @tbago in #1396
* SampleRequest写错了应为SampledRequest by @guodongxiaren in #1420
* 修复rpc_replay中ChannelGroup初始化时resize的bug by @guodongxiaren in #1422
* fix warning in gcc8+ by @stdpain in #1381
* 文档修复：BRPC_RPC_VALIDATE_GFLAG改为BRPC_VALIDATE_GFLAG by @guodongxiaren in #1426
* 消除高版本GCC上编译时大量-Wclass-memaccess的警告 by @guodongxiaren in #1427
* rm DISCLAIMER-WIP and use DISCLAIMER for 1.0 release by @tanzhongyi003 in #1432
* Update flat_map.h by @yanjianglu in #1352
* add flag BTHREAD_NEVER_QUIT by @ustccy in #1193
* implement weighted randomized load balancer #1254 by @serverglen in #1314
* rpc_view support setting timeout by @serverglen in #1459
* Fix baidu_rpc_protocol.cpp a variable incorrectly named by @serverglen in #1451
* Fix Socket::WaitAndReset memory leak by @wwbmmm in #1456
* docs: fix server_push.md by @lorinlee in #1463
* Add gitignore for files generated during test by @wwbmmm in #1473
* docs: fix broken link in CONTRIBUTING.md by @lorinlee in #1481
* Add FlatMap example code by @serverglen in #1484
* Fix apache thrift download failed by @wwbmmm in #1495
* Fix http client doc by @wwbmmm in #1493
* Add DomainListNamingService which resolves services from the given list periodically by @chenzhangyi in #1509
* check is valid character in uri by @guodongxiaren in #1506
* emplace/emplace_back replace insert/push_back for pair by @guodongxiaren in #1504
* Update gdb_bthread_stack.py by @tanzhongyi003 in #1521
* Update get_brpc_revision.sh by @tanzhongyi003 in #1520
* Tanslate getting_started.md into Chinese by @guodongxiaren in #1515
* Update NOTICE by @tanzhongyi003 in #1519
* add missing notice for tools/pprof, test/crc32c_unittest.cc by @tanzhongyi003 in #1522
* docs: fix centos compile options by @lorinlee in #1525
* Add rtmp/FlvWriterOptions to support writing audio/video content only by @v1siuol in #1505
* upgrade baseimage to 20.04 by @tanzhongyi003 in #1531
* release 1.0.0-rc02 by @lorinlee in #1538
* Fix tools/get_brpc_revision.sh by @v1siuol in #1536
* update docs by @serverglen in #1499
* Set hostname rather than ip when channel Init by hostname but not host in http_requst().uri() by @guodongxiaren in #1529
* add cases.md by @tanzhongyi003 in #1551
* Add use case of Baidu by @wwbmmm in #1557
* docs(circuit_breaker): add ema wiki link by @JiaoZiLang in #1575
* Add Case by @guodongxiaren in #1576
* community: add release doc by @lorinlee in #1582
* Update release_cn.md by @tanzhongyi003 in #1588

Thanks to all contributors for the 1.0.0 version!