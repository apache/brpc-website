---
title: "bRPC 1.5.0"
linkTitle: "bRPC 1.5.0"
weight: 5
date: 2023-05-07
description: >
  Apache bRPC 1.5.0 Release Version.
---
## What's Changed

### Feature:
- Support IPv6 for DNS resolve by @jsl422 in #2139
- Support naming timer sampling and worker threads by @ehds in #2136
- Support different TimeoutConcurrencyConf for different method by
@yanglimingcn in #2112
- Add bvar is_hidden by @serverglen in #2205
- Add server concurrency in status builtin service by @chenBright in #2097
- Add avg latency for prometheus metrics by @Huixxi in #2024

### Bugfix:
- Fix the issue of const unused in the example of RDMA by @goldenbean in #2187
- Fix domain naming service host name buffer length by @ehds in #2179
- Fix memory leak of socket by @chenBright #2169
- Fix not end wait when ns fails to start by @chenBright #2162
- Fix ci failed with wrong path of libprotoc by @guodongxiaren in #2132
- Fix the periodic naming service quit problem by @chenBright in #2123

### Enhancement:
- Remove wordexp by @wwbmmm in #2218
- Update github workflows to skip builds for markdown-file-only
changes by @kiminno in #2175
- Reject initializing FlatMap when nbucket is 0 by @jamesge
- Optimize some codes that violates the C++ One Definition Rule
[-Wodr] by @lrita in #2161
- Add _Alloc template parameters for FlatMap and FlatSet by @old-bear in #2149
- Add type BasicStringPiece::const_pointer by @lrita in #2141
- Operator overloading of PtrContainer by @chenBright in #2107
- Make sure we can receive at least one request @yanglimingcn in #2106
- Reduce cpu overhead when using rdma by @Tuvie in #2100

### Others
- Prefer to use env to find bash by @wasphin
Improve/add documents by @haihuju, @tanzhongyi003, @wwbmmm, @wasphin,
@maheshrjl, @chenBright, @NIGHTFIGHTING, @Huixxi, @zuyu, @kiminno,
@wy1433, @20083017, @Thunderbrook

Full Changelog can be found at: https://github.com/apache/brpc/compare/1.4.0...1.5.0

Thanks to all contributors for the 1.5.0 version!