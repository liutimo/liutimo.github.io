---
uuid: 248a6ce0-8d26-11ef-87e9-758771b60cf9
title: rgw data layout
date: 2024-10-17 10:07:18
updated:
tags:
categories:
---

# 用户

> 元数据池

| 命名空间 | 对象名 | 保存内容|
| --- | --- | --- |
| users.keys| access_key | `RGWUID` |
| users.email| email | `RGWUID` |
| users.uid| uid | `RGWUID` + `RGWUserInfo` |


