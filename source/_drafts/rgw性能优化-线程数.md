---
uuid: 248a6ce3-8d26-11ef-87e9-758771b60cf9
title: rgw性能优化--线程数
date: 2024-10-14 10:57:01
updated:
tags: rgw
categories: ceph
---

在rgw中，通过`rgw_thread_pool_size`来控制工作线程的数量，该配置项的默认值为512。

75K对象

| rgw_thread_pool_size |  是否协程 | 时延 | IOPS | 
|---|---|---|---|
|1024|否|21.16 ms|1130.45 op/s|
|512|否|21.5 ms|1112.88 op/s|
|256|否|21.74 ms|1101.01 op/s|
|64|否|22.5 ms|1063.26 op/s	|
|16|否|26.44 ms|905.48 op/s|
|16|是|21.25 ms|1126.82 op/s|

