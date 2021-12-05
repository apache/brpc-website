---
layout: page
title: Downloads
description: Project Downloads page
group: nav-right
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
{% include JB/setup %}

## {{ site.data.project.name }} Downloads

{{ site.data.project.name }} is released as a source artifact.
We are still working on our first apache release, 
hopefully it will be done in Oct 2019. 


### Release Candidate 
<!--when pass vote, we can change it back to Release Artifacts
-->
<table class="table table-hover sortable">
    <thead>
        <tr>
            <th><b>Name</b></th>
            <th><b>Archive</b></th>
            <th><b>SHA-512</b></th>
            <th><b>Signature</b></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>{{ site.data.project.name }} {{site.data.project.latest_release}} (tar.gz)</td>
            <td><a href="https://www.apache.org/dyn/closer.lua/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0/apache-brpc-1.0.0-incubating-src.tar.gz.asc">ASC</a></td>
        </tr>
        <tr>
            <td>{{ site.data.project.name }} 1.0.0-rc02 (tar.gz)</td>
            <td><a href="https://www.apache.org/dyn/closer.lua/incubator/brpc/1.0.0-rc02/apache-brpc-1.0.0-rc02-incubating-src.tar.gz">tar.gz</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0-rc02/apache-brpc-1.0.0-rc02-incubating-src.tar.gz.sha512">SHA-512</a></td>
            <td><a href="https://downloads.apache.org/incubator/brpc/1.0.0-rc02/apache-brpc-1.0.0-rc02-incubating-src.tar.gz.asc">ASC</a></td>
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

Choose a source distribution in either *tar* or *zip* format,
and [verify](https://www.apache.org/dyn/closer.cgi#verify)
using the corresponding *pgp* signature (using the committer file in
[KEYS](https://downloads.apache.org/{{ site.data.project.incubator_slash_name }}/KEYS)).
If you cannot do that, the *md5* hash file may be used to check that the
download has completed OK.

For fast downloads, current source distributions are hosted on mirror servers;
older source distributions are in the
[archive](https://archive.apache.org/dist/{{ site.data.project.incubator_slash_name }}/).
If a download from a mirror fails, retry, and the second download will likely
succeed.

For security, hash and signature files are always hosted at
[Apache](https://www.apache.org).

