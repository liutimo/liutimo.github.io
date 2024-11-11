---
uuid: 588ca4d0-a01e-11ef-a313-57445f10154d
title: RGW桶分片随笔
banner_img: 'https://bing.mcloc.cn/img/2024/11/11/2024-11-11_hd.jpg'
index_img: /imgs/common/timo.png
abbrlink: 26286
date: 2024-11-11 19:16:06
updated:
tags:
categories:
---




1. 创建桶分片
   
    创建桶时，会创建桶分片。
    RGWRados::create_bucket --> RGWSI_BucketIndex::init_index
    命名规则：
    dir_oid_prefix + bucket_id + shard_idx。

    下发请求： create + cls(RGW_BUCKET_INIT_INDEX)。
    ```c++
    int rgw_bucket_init_index(cls_method_context_t hctx, bufferlist *in, bufferlist *out)
    {
      CLS_LOG(10, "entered %s", __func__);
      bufferlist header_bl;
      int rc = cls_cxx_map_read_header(hctx, &header_bl);
      if (rc < 0) {
        switch (rc) {
        case -ENODATA:
        case -ENOENT:
          break;
        default:
          return rc;
        }
      }

      if (header_bl.length() != 0) {
        CLS_LOG(1, "ERROR: index already initialized\n");
        return -EINVAL;
      }

      rgw_bucket_dir dir;

      return write_bucket_header(hctx, &dir.header);
    }
    ```
2. xxx