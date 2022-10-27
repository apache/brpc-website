---
title: "bRPC 1.3.0"
linkTitle: "bRPC 1.3.0"
weight: 1
date: 2022-10-25
description: >
  Apache bRPC 1.3.0 Release Version.
---
## What's Changed
New features:
- Support gcc on linux arm64 by @jamesge
- Support gcc version >= 11.2.0 by @wwbmmm in #1783
- Support the latest version of bazel(default v4.2.2) by @hcoona in #1657
- Support multi-dimension bvar, a powerful extension of bvar by @serverglen in #1608
- Restruct event_dispatcher source file by @guodongxiaren in #1888
- Add http retry with error code policy by @chenBright in #1927
- Add Nacos naming service by @yyweii in #1922
- Add customized server bvar prefix by @jenrryyou in #1854
- Add escape log content before printing by @jamesge

Bugfix:
- Fix issues in FlatMap by @jamesge
- Fix override issue in pb by @jamesge, @wwbmmm
- Fix ALIGNAS/ALIGNOF/BAIDU_CACHELINE_ALIGMENT by @jamesge
- Fix some warnings for clang and revert changes on ALIGNAS/ALIGNOF by @jamesge
- Fix rpc_replay continue when failed to init channel by @ehds in #1938
- Fix multi-dimension bvar compile error by @dabao085 in #1937
- Fix bvar_dump_tabs default value problem by @yyweii in #1920
- Fix butex_wait failed with timeout by @Huixxi in #1917
- Fix rpc_replay can't send request equably by @bumingchun in #1910
- Fix compile warning due to DumpOptions object by @ml-haha in #1905
- Fix test_bvar fail on m1 mac by @wwbmmm in #1901
- Fix the slow test in brpc_socket_unittest.cpp by @zyearn in #1898
- Fix the first bthread keytable on worker pthread will be deleted twice by @chenBright in #1884
- Fix currently broken MacOS build by @zyearn in #1871
- Fix ProcessHttpRequest supports for http2 by @dandyhuang in #1868
- Fix get_value core caused by the sampler thread start too early by @Huixxi in #1863
- Fix UDS ut failed on MacOS by @wwbmmm in #1843
- Fix coredump cause by bad growth_non_responsive http request by @acelyc111 in #1278
- Fix not to abort when checking the errorno with unicode string by @tobegit3hub in #1142

Other improvements:
- Improve/add documents by @wwbmmm, @JackBoosY, @morningman, @serverglen, @chenBright, @guodongxiaren, @xdh0817, @KaneVV1, @tanzhongyi003, @lzfhust, @Huixxi
- Fix typos by @opheliaKyouko, @day253, @chenBright, @fansehep

Thanks to all contributors for the 1.3.0 version!