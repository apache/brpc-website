---
title: "下载"
linkTitle: "下载"
weight: 2
menu:
  main:
    weight: 20
aliases:
- "/zh/download"
- "/zh/downloadbrpc"
- "/zh/docs/downloadbrpc"
date: 2021-08-12
description: >
  下载bRPC发行版。
---
<!--
{% comment %}
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endcomment %}
-->
## 下载 Apache bRPC (incubating)

Apache bRPC（孵化）作为源工件发布。我们很高兴宣布我们的1.3.0版本已经发布如下了！


### 候选版本
<!--when pass vote, we can change it back to Release Artifacts
-->
<table class="table table-hover sortable">
    <thead>
        <tr>
            <th><b>名字</b></th>
            <th><b>存档</b></th>
            <th><b>加密算法</b></th>
            <th><b>签名</b></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Apache bRPC (incubating) 1.3.0 (tar.gz)</td>
            <td><a href="https://dlcdn.apache.org/incubator/brpc/1.3.0/apache-brpc-1.3.0-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.3.0/apache-brpc-1.3.0-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.3.0/apache-brpc-1.3.0-incubating-src.tar.gz.asc">ASC</a></td>
        </tr>
        <tr>
            <td>Apache bRPC (incubating) 1.2.0 (tar.gz)</td>
            <td><a href="https://dlcdn.apache.org/incubator/brpc/1.2.0/apache-brpc-1.2.0-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.2.0/apache-brpc-1.2.0-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.2.0/apache-brpc-1.2.0-incubating-src.tar.gz.asc">ASC</a></td>
        </tr>
        <tr>
            <td>Apache bRPC (incubating) 1.1.0 (tar.gz)</td>
            <td><a href="https://dlcdn.apache.org/incubator/brpc/1.1.0/apache-brpc-1.1.0-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.1.0/apache-brpc-1.1.0-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.1.0/apache-brpc-1.1.0-incubating-src.tar.gz.asc">ASC</a></td>
        </tr>
        <tr>
            <td>Apache bRPC (incubating) 1.0.0 (tar.gz)</td>
            <td><a href="https://dlcdn.apache.org/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz.asc">ASC</a></td>
        </tr>
        <!--tr>
            <td>Release Notes</td>
            <td><a href="/releases/spark/{{ site.data.project.latest_release }}/release-notes">{{ site.data.project.latest_release }}</a></td>
            <td></td>
            <td></td>
            <td></td>
        </tr-->
    </tbody>
</table>

选择*tar*或*zip*格式的源发行版，并使用相应的*pgp*签名（使用[密钥](https://downloads.apache.org/incubator/brpc/KEYS)中的提交者文件）进行[验证](https://www.apache.org/dyn/closer.cgi#verify)。如果您无法做到这一点，可以使用*md5*哈希文件检查下载是否已完成。

为了快速下载，当前的源发行版托管在镜像服务器上；较旧的源发行版在[存档](https://archive.apache.org/dist/incubator/brpc/)中。如果从镜像下载失败，请重试，第二次下载可能会成功。

为了安全起见，哈希文件和签名文件始终托管在[Apache](https://www.apache.org)上。
