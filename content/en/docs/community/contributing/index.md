---
title: "Contribute Guide"
linkTitle: "Contribute Guide"
weight: 3
date: 2022-09-14
description: >
  Contribute to bRPC
---
If you meet any problem or request a new feature, you're welcome to [create an issue](https://github.com/brpc/brpc/issues/new/choose).

If you can solve any of [the issues](https://github.com/brpc/brpc/issues), you're welcome to send the PR to us.

Before the PR:

* Fully comply with the [ASF Code of Conduct](https://www.apache.org/foundation/policies/conduct.html).
* Make sure your code style conforms to [google C++ coding style](https://google.github.io/styleguide/cppguide.html). Indentation is preferred to be 4 spaces.
* The code appears where it should be. For example the code to support an extra protocol should not be put in general classes like server.cpp, channel.cpp, while a general modification would better not be hidden inside a very specific protocol.
* Has unittests.

After the PR:

* Make sure the [travis-ci](https://app.travis-ci.com/github/apache/incubator-brpc/pull_requests) passed.