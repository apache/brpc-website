---
title: "bRPC 1.4.0"
linkTitle: "bRPC 1.4.0"
weight: 6
date: 2023-02-07
description: >
  Apache bRPC 1.4.0 Release Version.
---
## What's Changed
New features:
- Support RDMA by @Tuvie in #1836, #1967, #2005 and #2036
- Support to dump MVariable in Prometheus format by @ldak4747 in #1964
- Support to limit max bytes in stream consume queue with the same host socket by @chenbay in #1958
- Add timeout concurrency limiter by @yanglimingcn in #2027
- Add a flag to manage http body on error by @jamesge
- Add gflag to disable the sampler thread by @leaf-potato in #1990
- Add delete_stats and has_stats interface to MVariable by @serverglen in #2041
- Optimize parallel channel request map method by @cdjingit in #2057

Bugfix:
- Fix the linkage errors caused by duplicate symbols by @adonis0147 in #1936
- Fix "sched_to itself" error when building by Clang on Linux aarch64 by @adonis0147 in #1950
- Fix arena cleared early when parsing redis message by @dorothy00dd2 in #1959
- Fix HTTP invalid host issue for channel not inited by http(s) by @thorneliu in #1973
- Fix MacOS warning: bool literal returned from 'main' -Wmain by @zyearn in #2020
- Fix issue of ssl error code by @yyweii in #2019
- Fix: domain name length by @wayslog in #1965
- Fix demangle core by @wwbmmm in #2037

Enhancement
- Add rdma_performance bazel support by @372046933 in #1984
- Add bazel third_party support by @fansehep in #1996
- Fall back to libibverbs.so.1 by @372046933 in #1985
- Refresh write timestamp when returning a Socket to its pool; change default value of -idle_timeout_second to 30. by @jamesge
- Early return for IOBuf::append_user_data when size == 0 by @372046933 in #2009
- Make BUTIL_RAPIDJSON_NAMESPACE_BEGIN::GenericDocument's handler method public to enable outside custom parser/writer by @old-bear in #2026
- Only allow to convert root array to single repeated pb by @chenBright in #2035
- FlatMap's value supports unique_ptr by @jamesge

Others
- Fix warning message error by @leaf-potato in #1989
- Migrate to GitHub workflow by @guodongxiaren and @zyearn in #1899, #2008, #2015, #2018, #2023 and #2030
- Reduce UT log output by @wwbmmm
- Support to pack rpm for RHEL 9 distributions by @wasphin in #1955
- Avoid std::string copy @ml-haha in #1969
- Include directly dependent header by @372046933 in #1993
- Update warning message on iobuf.cpp by @wwbmmm
- Remove unnecessary semicolon by @guodongxiaren in #2004
- Add a description to the profile graph by @hongliuliao in #2007
- Delete deprecated logoff_ms gflag in example folder by @leaf-potato in #2064
- Fix rpc maybe error: "libbrpc.so: undefined symbol: pthread_mutex_lock" by @co0l1ce in #2049 and roll-backed in #2076
- Keep bthread TaskGroup abi compatible with NDEBUG macro by @wwbmmm in #2047
- Improve/add documents by @fansehep, @cuishuang, @tanzhongyi003, @lorinlee, @Huixxi, @steven-66, @serverglen, @wwbmmm, @wasphin, @Tuvie, @0xflotus, @thinh2, @leaf-potato, @TousakaRin, @cdjingit, @chenBright, @freemandealer and @yanglimingcn

Thanks to all contributors for the 1.4.0 version!