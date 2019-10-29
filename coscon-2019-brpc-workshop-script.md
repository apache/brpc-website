# 介绍
* 这是开源社组织的2019年开源年会上brpc workshop所使用的脚本

# 课程介绍：
  * rpc & brpc
  * setup & 第一个程序
  * 第二个程序

# 面向对象：
* 对rpc有一定了解的C++程序员


# 要求
* 学员使用本人的笔记本进行上机练习
* 笔记本上docker已经安装并设置完毕
* 笔记本能连互联网

# 准备工作
* 下载brpc源码
  *  git clone https://github.com/apache/incubator-brpc
* build docker image
  * cd incubator-brpc
  * docker build -t brpc:0.9.7rc1 .
  * docker image ls |grep brpc
* 运行docker image
  * docker run -it brpc:0.9.7rc1 /bin/bash


# 第一个rpc程序
  * server 端
    * docker run -p 8000:8000 -it brpc:0.9.7rc1 /bin/bash
      * cd /brpc/example/echo_c++
      * make
      * ./echo_server
  * 另起一个终端，作为client
    * docker run --network=host -it brpc:0.9.7rc1 /bin/bash
      * cd /brpc/example/echo_c++
      * make
      * ./echo_client

# 进阶程序
