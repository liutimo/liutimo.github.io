---
uuid: 248a6ce4-8d26-11ef-87e9-758771b60cf9
title: rgw性能优化
date: 2024-09-20 11:11:09
updated:
tags: rgw
categories: ceph
---

仅从rgw的视角开始分析，读写涉及如下几个点：
1. 元数据
2. 数据
3. 鉴权
4. ...

如何优化rgw的性能。

## 元数据
元数据裁剪。以acl为例，我们每次上传对象都会设置ACL的xattr，其真的有必要吗？


## 索引

上传对象时，我们需要读取并修改对象索引信息。尝试缓存在cls层缓存这一部分数据。用以加速？

prepare index:

1. RGW_GUARD_BUCKET_RESHARDING
2. RGW_BUCKET_PREPARE_OP

## 数据
数据大小我们无法裁剪，只能通过优化 数据处理流程来寻找优化空间。


## 鉴权
鉴权流程优化