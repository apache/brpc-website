---
title: "bRPC 1.2.0"
linkTitle: "bRPC 1.2.0"
weight: 2
date: 2022-08-07
description: >
  Apache bRPC 1.2.0 Release Version.
---
## What's Changed
New features:
   - Support apple silicon by @jamesge
   - Add an option to allow serialize/deserialize to/from a json array by @wasphin in #1604
   - Add redis empty request check by @lzfhust in #1745
   - Add butex_wake_all support nosignal flag by @yanglimingcn in #1751
   - Add wr/wrr policy degradation by @Huixxi in #1571
   - Add with_snappy in cmake by @renzhong in #1799
   - Add check append return code by @wwbmmm in #1762
   - Add CXXFLAGS in compiling protoc-gen-mcpack by @jamesge

Bugfix:
   - Fix the header guard of brpc/periodic_task.h by @TousakaRin in #1820
   - Fix build warning, ByteSize() is deprecated, use ByteSizeLong() instead by @yangzhg in #1723
   - Fix c struct compile error by @wolfdan666 in #1736
   - Fix auto https check by @renzhong in #1754
   - Fix json2pb::JsonToProtoMessage() supports parsing multiple jsons by @jamesge
   - Fix compile error due to std limits header absent by @GOGOYAO in #1764
   - Fix a deadlock happened in ClearAbandonedStreamsImpl path by @zyearn in #1781
   - Fix send WindowUpdate when ClearAbandonedStreams is called by @zyearn in #1786
   - Fix _dl_sym undefined reference by @wwbmmm in #1784
   - Fix thrift protocol exception by @lzfhust in #1790
   - Fix Brpc build tools error when only build static by @stdpain in #1797
   - Fix discovery naming service core by @serverglen in #1802
   - Fix rpc_press can't send request equably by @bumingchun #1763
   - Fix hostname2ip fails when aux_buf is not long enough by @chenBright in #1818

Other improvements:
   - Improve documents by @wwbmmm, @tanzhongyi003, @guodongxiaren, @372046933, @wasphin, @hawkxiang, @TousakaRin, @Huixxi, @serverglen
   - Fix typos by @jamesge, @cyberkillor

Thanks to all contributors for the 1.2.0 version!
