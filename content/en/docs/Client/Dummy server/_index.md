---
title: "Dummy server"
linkTitle: "Dummy server"
weight: 14
date: 2021-08-12
description: >
  Learn how to use dummy server.
---
If your program only uses client in brpc or doesn't use brpc at all, but you also want to use built-in services in brpc. The thing you should do is to start an empty server, which is called **dummy server**.

# client in brpc is used

Create a file named dummy_server.port which contains a port number(such as 8888) in the running directory of program, a dummy server would be started at this port. All of the bvar in the same process can be seen by visiting its built-in service.
![img](/images/docs/dummy_server_1.png) ![img](/images/docs/dummy_server_2.png) 

![img](/images/docs/dummy_server_3.png)

# brpc is not used at all

You must manually add the dummy server. First read [Getting Started](../../getting_started/) to learn how to download and compile brpc, and then add the following code snippet at the program entry:

```c++
#include <brpc/server.h>
 
...
 
int main() {
    ...
    brpc::StartDummyServerAt(8888/*port*/);
    ...
}
```