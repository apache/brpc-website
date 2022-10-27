---
title: "bRPC 1.1.0"
linkTitle: "bRPC 1.1.0"
weight: 3
date: 2022-04-12
description: >
  Apache bRPC 1.1.0 Release Version.
---
## What's Changed
New features:
* Support ipv6 and unix domain socket by @wwbmmm in #1560
* Support protobuf 3.19.x by @hcoona in #1679
* Support http protocol dump and replay by @guodongxiaren in #1503
* Support nshead protocol dump and replay by @wwbmmm in #1486
* Support parse proto-text format http request body by @hiberabyss in #1690
* Support deliver timeout from client to server for baidu_std protocol by @wwbmmm in #1472
* Support inherit span on bthread create by @wwbmmm in #1494
* Add rpc_replay BUILD file by @taoxu in #1677
* Add brpc server Start(PortRange, const ServerOptions*) by @serverglen in #1460
* Add FlatMap insert(const std::pair<key_type, mapped_type>& kv) by @serverglen in #1468
* Add server eps bvar @serverglen in #1483

Bugfix:
* Fix CheckHealth not set has_request_code bug by @serverglen in #1502
* Fix a bug that server will send unexpected data frame to client if there are errors occur during processing stream create request by @jenrryyou in #1516
* Fix LA selection runs too long by @KaneVV1 in #1567
* Fix HttpResponse error by @jl2005 in #1620
* Fix bvar status compile error by @zwkno1 in #1625
* Fix InputMessenger client side retry policy by @ehds in #1680
* Fix work_stealing_queue_unittest for ARM by @TKONIY in #1709
* Fix LatencyRecorder qps not accurate by @wwbmmm in #1708
* Fix compile error after gcc11 with --std=c++20 by @hiberabyss in #1719
* Fix unstable UT link error by @wwbmmm in #1711
* Fix Thrift download url to avoid pr build failed by @yangzhg in #1725
* Remove grpc ParseH2Settings warning log by @yanjianglu in #1599

Other improvements:
* Improve documents by @wwc7654321, @wwbmmm, @tanzhongyi003, @mahongweichina, @cdjingit, @dl239, @ehds
* Fix typos by @yangzhg, @egolearner, @PengyiPan, @Aaaaaaron, @ehds, @JiaoZiLang, @mapleFU

Thanks to all contributors for the 1.1.0 version!