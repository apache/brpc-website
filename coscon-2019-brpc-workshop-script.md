# 介绍
* 这是开源社组织的2019年开源年会上brpc workshop所使用的脚本

# 课程介绍：
* rpc & brpc
* setup 
* 三个实际示例

# 面向对象：
* 对rpc有一定了解的C++程序员
* 使用过docker（或有Linux环境）

# 要求
* 学员使用本人的笔记本进行上机练习
* 笔记本上docker已经安装并设置完毕
* 笔记本能连互联网

# 准备工作

## Step1 下载brpc Dockerfile
* mkdir brpc && cd brpc
* wget https://raw.githubusercontent.com/apache/incubator-brpc/master/Dockerfile > Dockerfile

## Step2 本机build镜像(recommand) or 直接拉现成镜像

### 本机build镜像
* docker build -t brpc:0.9.7rc1 .

这一步根据网络环境不同而耗时不同，一般需要若干分钟。

### 直接拉现成镜像
* docker pull 13718272827/brpc:0.9.7rc1

## Step3 运行Container

* docker run -it brpc:0.9.7rc1 /bin/bash

或者

* docker run -it 13718272827/brpc:0.9.7rc1 /bin/bash

如果出现bash界面，则顺利进入装有brpc的container。

# 第一个rpc场景（echo）

server 端
* docker run -p 8000:8000 -it brpc:0.9.7rc1 /bin/bash
* (docker run -p 8000:8000 -it 13718272827/brpc:0.9.7rc1 /bin/bash)
  * cd /brpc/example/echo_c++
  * make
  * ./echo_server

若看到`Server[XXXX] is serving on port=8000`日志，则说明server启动成功。

另起一个终端，作为client
* docker run --network=host -it brpc:0.9.7rc1 /bin/bash
* (docker run --network=host -it 1371827287/brpc:0.9.7rc1 /bin/bash)
  * cd /brpc/example/echo_c++
  * make
  * ./echo_client

若在server端看到交互日志，则echo程序运行成功。

# 进阶程序（加入负载均衡）
在这个例子中，将启动两台server，然后client通过负载均衡来访问这两台server。

server端第一台server：
* docker run -p 8000:8000 -it brpc:0.9.7rc1 /bin/bash
* (docker run -p 8000:8000 -it 13718272827/brpc:0.9.7rc1 /bin/bash)
    * cd /brpc/example/echo_c++
    * make
    * ./echo_server

server端第二台server：
* docker run -p 8001:8000 -it brpc:0.9.7rc1 /bin/bash
* (docker run -p 8001:8000 -it 13718272827/brpc:0.9.7rc1 /bin/bash)
    * cd /brpc/example/echo_c++
    * make
    * ./echo_server

上述命令中，起了两个server分别监听在本机的8000和8001端口。

client端
* docker run --network=host -it brpc:0.9.7rc1 /bin/bash
* (docker run --network=host -it 1371827287/brpc:0.9.7rc1 /bin/bash)
  * cd /brpc/example/echo_c++
  * make
  * ./echo_client --server=list://127.0.0.1:8000,127.0.0.1:8001 --load_balancer=rr

上述命令中，client会发送请求给127.0.0.1:8000,127.0.0.1:8001这两台机器，负载均衡算法是rr(roundrobin)，其余可选项为wrr，random，locality-aware(la)，c_murmurhash。

# 第三个rpc场景（流式传输）

server 端
* docker run -p 8001:8001 -it brpc:0.9.7rc1 /bin/bash
* (docker run -p 8001:8001 -it 13718272827/brpc:0.9.7rc1 /bin/bash)
  * cd /brpc/example/streaming_echo_c++/
  * make
  * ./echo_server

另起一个终端，作为client
* docker run --network=host -it brpc:0.9.7rc1 /bin/bash
* (docker run --network=host -it 1371827287/brpc:0.9.7rc1 /bin/bash)
  * cd /brpc/example/streaming_echo_c++/
  * make
  * ./echo_client
