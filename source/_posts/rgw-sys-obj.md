---
uuid: 3e8fcf20-aafb-11ef-8c5f-c103cc0f58d8
title: RGW Services -- sys obj
banner_img: 'https://bing.mcloc.cn/img/2024/11/25/2024-11-25_hd.jpg'
index_img: 'https://bing.mcloc.cn/img/2024/11/25/2024-11-25_hd.jpg'
abbrlink: 62799
date: 2024-11-25 15:02:33
updated:
tags: 
categories: rgw
---

## 什么是SysObj

System Object，指那些RGW中用于表示 S3/Swift中的一些概念的对象。如桶、用户、多站点等信息。
这些对象占据的空间不大，且创建后几乎不太会修改。

RGW为这些对象提供了统一的操作接口。其基本的使用方式如下:
```C++
RGWSysObjectCtx obj_ctx = sysobj_svc->init_obj_ctx();
RGWSI_SysObj::Obj sysobj = sysobj_svc->get_obj(obj_ctx, rgw_ra_obj(pool, oid));
// RGWSI_SysObj::Obj sysobj = obj_ctx.get_obj(rgw_ra_obj(pool, oid));

sysobj.rop().xxx(); // ROP, 对象数据/属性 get
// sysobj.wop().xxx();  // WOP, 对象数据/属性 set
// sysobj.omap().xxx(); // OmapOp, omap set/get
// sysobj.wn().xxx();  // WNOP, Watch Notify
```

> 额，我实在是看不懂RGWSysObjectCtx的作用，所以，我去看了一下reef版本的代码，发现这个东西已经被移除了。

所以，在reef版本中，其使用更加简单了：
```c++
RGWSI_SysObj::Obj sysobj = sysobj_svc->get_obj(rgw_ra_obj(pool, oid));

sysobj.rop().xxx(); // ROP, 对象数据/属性 get
// sysobj.wop().xxx();  // WOP, 对象数据/属性 set
// sysobj.omap().xxx(); // OmapOp, omap set/get
// sysobj.wn().xxx();  // WNOP, Watch Notify
```


RGWSI_SysObj抽象出Obj和Pool两个内部类，Obj用于操作对象的数据和元数据(omap/xattrs), Pool用于列举池对象。