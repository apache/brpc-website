---
title: "brpc初学者指南"
linkTitle: "brpc初学者指南"
weight: 2
date: 2021-11-30
---
(作者简介：lorinlee，是brpc新进committer，在字节跳动负责图数据库相关工作）

Apache bRPC(incubating)  是一款优秀的工业级 C++ RPC 框架，其兼具高性能、兼容多种协议、周边工具完善等多种优点于一身，已在国内多个大厂广泛使用。

有不少同学对brpc感兴趣，但还没有找到合适的路径，不知道如何学习。笔者是来自字节跳动的研发工程师，也曾遇到相同的困扰，后来通过自己的一些摸索，成为了brpc committer，期间也总结了一些经验，希望对大家有所帮助。

因此本文期望结合作者的经历，通过一些简单可行的步骤，帮助大家迈出探索brpc的脚步，学习它的设计与实现方式，以对大家日常的学习和工作有所帮助。也欢迎大家在熟悉brpc之后，参与到brpc的贡献中，比如文档完善、解决issue、bugfix、feature开发等等。

## 文档
初始接触brpc，文档是最好的入手材料。brpc的文档在国内开源项目中广受赞誉，原因是其不仅介绍了丰富的技术知识，还蕴含了设计者深入的思考与选择。

首先推荐阅读：[bRPC Overview](https://brpc.apache.org/docs/overview/)，这篇文档不仅综述性地介绍了brpc，它更是一篇索引，关键的技术都有超链接至对应的细节文档，阅读完这篇文档以及所有的链接，将能够对brpc有完整的认识。

另外，有些文档虽然在上面链接中已经涵盖了，但我依然想在这里再单独分类推荐一下：

1. IO：RPC框架作为网络通信框架，IO模型是非常重要的一环，其定义了客户端如何发送消息，服务端如何接受消息，这篇文档详细介绍了brpc对于这一过程的实现。常见的RPC框架会区分IO线程和Worker线程，但这样的设计是有弊端的。brpc则不对线程做区分，IO逻辑和业务逻辑全都在bthread中，通过调度器平衡任务在线程之间的负载。另外，brpc支持多种协议，并支持灵活的拓展，可以参考 protocol。

2. 线程模型，bthread：RPC框架另一个重要的部分是线程模型，brpc自研了用户态调度的M:N线程库bthread，既能达到优异的性能，又不会损失代码的可维护性。

3. bthread_id：这里的bthread_id并不是bthread本身的id（bthread_t），而是用于对同一个request的多次重试、超时处理等并发事件进行同步，以及在连接复用（单连接）的场景，维护request和response的映射。

4. IOBuf：RPC框架需要处理大量接收的消息、发送的消息，其中消息占用的内存分配在很多场景都可能会成为一大开销，brpc通过IOBuf减少内存分配、拷贝等开销。

5. Timer：RPC场景需要处理大量的时间事件，主要是对于超时的控制，比如对于一个请求，需要在其超时后及时终止并结束。一般的系统在吞吐上升的过程中，Timer的压力会不断增加，其拓展性会成为性能瓶颈。而brpc对Timer做了细致的设计，使timer操作几乎对RPC性能没有影响。

6. MemoryManagement：brpc实现了内存池对一些对象进行管理以提升性能，并且可以对这些对象附加version，简化了共享对象的回收逻辑。

7. bvar：系统服务通常需要采集大量的指标，用于系统监控、性能优化等，但通常大量的指标采集会严重影响性能，有些做法是通过采样来降低开销，但会导致指标不够准确。brpc的bvar则能够高效的收集大量指标。

## 源码
文档能够快速了解系统的核心设计思路，但如果要深入理解系统，还是需要对源码进行阅读。brpc是一款优秀的C++项目，其源码是值得大家阅读学习的。这里简单为大家把源码进行一些介绍，方便大家阅读。

整体的源码阅读过程，推荐先按照某些关键的执行流程来阅读，梳理其中的各个环节步骤，就能够理解每个模块在整个系统所处的位置；随后单独对重要的模块研读其实现，就能够比较好地掌握整个系统。

RPC框架一般会用在两个端，客户端和服务端，两者有些流程是共用的，有些是不同的。所以可以将简单的echo服务为出发点，分别对客户端和服务端的流程进行阅读。

### 客户端
对于客户端，典型的流程比如：客户端的初始化，发起RPC请求，处理应答，以及处理超时等逻辑。这里对一些关键的过程进行简述，大家可以在阅读过程中搜寻下述的一些关键函数名，找到代码对应的位置：

1. 客户端的初始化：Channel::Init，这里会做全局的初始化（GlobalInitializeOrDie），NamingService线程、LoadBalancer的初始化等。

2. 发起RPC请求：Channel::CallMethod，这里会分配call_id（bthread_id），序列化请求内容（_serialize_request -> Protocol::SerializeRequest），注册BackupRequest/Timeout的事件，选择要发送消息的Socket，打包协议内容（_pack_request -> Protocol::PackRequest），消息写入fd（Socket::Write）。

3. 处理应答：客户端在创建Socket时，会将对应的fd放在EventDispatcher中，并注册对应的回调函数（InputMessenger::OnNewMessages）。在Linux系统下，当fd有可读数据，EventDispatcher::Run中会通过epoll_wait拿到对应fd，调用Socket::StartInputEvent，并最终回调至InputMessenger::OnNewMessages。在InputMessenger::OnNewMessages中，会从fd中读取数据，按请求粒度切分数据，并将每个请求通过协议定义的处理函数中进行执行（Protocol::ProcessResponse），并最终调用至Controller::OnVersionedRPCReturned结束RPC，或执行重试等。

4. 处理超时：超时事件是注册在Timer线程中的，在RPC超时后，Timer线程会回调HandleTimeout，并经由bthread_id的机制回调至Controller::HandleSocketFailed，最终也会到达Controller::OnVersionedRPCReturned，在这里结束RPC或执行重试等。

### 服务端
对于服务端，典型的流程包括：服务的启动，收到RPC请求，执行处理，返回应答等逻辑。这里也对一些关键的过程进行简述，方便大家找到对应的代码位置：

1. 服务启动：Server::Start，这里会做一些初始化（Server::InitializeOnce），数据存储工厂初始化（SessionData，ThreadLocalData，bthread_local），创建Acceptor用于接受客户端连接。

2. 收到RPC请求：服务端收到的请求也是通过EventDispatcher进行派发的。在收到客户端连接请求时，EventDispatcher会回调Acceptor::OnNewConnections进行新连接建立，并将新连接也托管至EventDispatcher。在连接收到数据后，会由EventDispatcher回调至InputMessenger::OnNewMessages，从fd中读取数据，对数据按请求粒度进行切分，并调用协议中注册的处理函数（Protocol::ProcessRequest）进行处理。（客户端和服务端在这里注册的回调是不同的）。

3. 执行处理：请求的处理逻辑与协议有关，不过大体的逻辑是相似的，这里以baidu_std协议为例简述一下处理流程。baidu_std协议的请求处理逻辑在ProcessRpcRequest中，其会设置一些上下文信息，并发控制与限流，反序列化协议头和元数据，根据元数据找到具体的请求信息，反序列化请求体，注册RPC结束（done->Run()）的回调函数（SendRpcResponse），并调用用户注册的Service对应方法进行处理。

4. 返回应答：在用户处理函数处理结束后，通常会调用done->Run()结束RPC，此时会回调上面注册的函数SendRpcResponse，将用户写好的Response，连同协议头和元数据一起序列化，写入fd中。

### 源码目录
这里再对brpc的源码目录做个基本的介绍，方便大家有个整体认识。首先推荐大家阅读brpc中的示例，其能够帮助大家对brpc的使用场景和使用方法有所了解，并在后续对核心系统源码阅读的过程能够带着使用场景阅读。

example目录下，主要有几个目录：

1. echo_c++：同步客户端的echo示例

2. asynchronous_echo_c++：异步客户端的echo示例

3. backup_request_c++：客户端开启backup_request示例

4. cancel_c++：客户端取消rpc示例

5. parallel_echo_c++：客户端同时访问多个服务示例

6. partition_echo_c++：客户端分区调用的示例

7. dynamic_partition_echo_c++：客户端动态分区调用的示例

8. selective_echo_c++：客户端多Channel间负载均衡示例

9. multi_threaded_echo_c++：客户端多线程echo示例

10. multi_threaded_echo_fns_c++：客户端多线程+服务端多server示例

11. auto_concurrency_limiter：服务端开启自适应限流并验证其有效性

12. cascade_echo_c++：服务端级联调用示例

13. session_data_and_thread_local：服务端使用SessionData和ThreadLocalData

14. grpc_c++：grpc协议示例

15. http_c++：http协议示例

16. memcache_c++：memcache客户端示例

17. redis_c++：redis协议示例

18. thrift_extension_c++：thrift协议示例

19. streaming_echo_c++：大块数据传输示例


而brpc系统核心的实现在src目录中，其主要有几个目录：

1. brpc  
brpc目录下主要是RPC框架的实现，内容相对比较多，涵盖了网络通信框架、支持的各种协议、builtin服务等，以及包括负载均衡、服务发现、限流等。

2. bthread  
bthread目录下主要是用户态调度的M:N线程库的实现。其中包括bthread调度器、bthread同步原语、timer线程等。

3. butil  
butil目录下是丰富的工具库，包括了对文件、网络、时间、线程、内存、字符串、日志、容器等等各种基础组件的封装，以及一些第三方库。

4. bvar  
bvar目录下是指标收集的实现。

## 实践
俗话说：“纸上得来终觉浅，绝知此事要躬行”。在对源码有一定的阅读之后，就推荐大家具体地解决一些问题，这样能够对系统可以有更好地理解。

### Issue
首先可以解决github issue。brpc有很多的用户，用户会在使用过程中遇到不同的问题，并发布在github issue中。通过对这些issue的解答，不仅能够加深对系统的认识，并且能够帮助到其他人，是一件非常有意义的事情。

当然，大家在使用过程中遇到的问题，也欢迎提在issue中，比如对系统改进的一些想法、发现的一些bug等。issue对比群聊的好处在于之前的问题可以追溯，同样的问题讨论一遍之后便会有完整的记录，如果后来的人遇到类似的问题，可以直接查阅前面的issue自行解决。

此外，之前的issue中对一些问题的讨论，也是非常好的学习材料。其中有些issue依然没有结论，期待着大家参与交流。

brpc社区有很多的issue，对于issue的管理，会给一些issue打上对应的lebel，目前label的类型有如下几种：

1. bug：确认为bug的问题

2. discussion：讨论

3. enhancement：待改进的问题，需要优化

4. feature：新特性，需要开发

5. good first issue：适合brpc初学者解决的问题

6. help wanted：适合对brpc有一些了解的同学解决的问题

7. official：由brpc官方提出的问题

8. regression：兼容性问题

9. security：安全性问题

10. wontfix：不再修复的问题

初学的同学可以选择 "good first issue" 中的问题进行解决，以及 "help wanted" 中的也可以尝试。

### PR
另外的实践方式就是提PR了。任何对项目有益的PR，不论它改进程度的大小，都是非常好的。可做的比如编写/翻译文档、修复bug、开发feature、代码重构、性能优化、修复typo等等。

通过提PR，大家会实际动手参与到brpc的建设中，会对系统有更好地理解，并且会更广泛地帮助到所有使用brpc的用户；而brpc项目也会因大家的贡献而一点点进步。

如果想提PR，但对做什么有些迷茫，可以参考一些label下的issue，比如 "good first issue"，"help wanted" 等，都比较适合初学者。当慢慢熟悉后，可以尝试解决 "enhancement"，"official"，"feature"，"bug" 等label下的问题。

## 结语
brpc是一个优秀的C++项目，值得大家学习它的设计与实现。但brpc同时也是相对有些复杂的项目，其有13w+行代码，需要一定的学习过程。

本文推荐了大家一些学习的入手点，希望能帮助大家更好地入门，并在学习过程中有丰厚的收获。当然，也期待大家在熟悉brpc之后，能够参与到brpc的贡献中，帮助到大量brpc的使用者。
