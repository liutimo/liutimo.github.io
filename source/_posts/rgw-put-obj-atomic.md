---
uuid: 80eeedf0-8f81-11ef-87b8-b57bfc12ab55
title: RGW对象整体上传
abbrlink: 44522
date: 2024-10-21 15:53:04
updated: 
banner_img: https://bing.mcloc.cn/img/2024/10/24/2024-10-24_hd.jpg
index_img: /imgs/common/timo.png
tags: rgw
categories: ceph
mermaid: true
---


## 整体上传的对象的结构

```mermaid
---
title: put obj class diagram
config:
  theme: neutral
---
classDiagram
  class DataProcessor {
    <<interface>>
    + process(data, offset) int
  }
  class ObjectProcessor {
    <<interface>>
    + prepare() int
    + complete() int
  }
  ObjectProcessor --|> DataProcessor
  class Pipe {
    - DataProcessor* next
  }

  Pipe --|> DataProcessor

  class ChunkProcessor {
    - uint64_t chunk_size
    - bufferlist chunk
    + process(data, offset) int
  }

  class StripeProcessor {
    - StripeGenator* gen
    - pair<uint64_t, uint64_t> bounds
  }

  ChunkProcessor --|> Pipe
  StripeProcessor --|> Pipe

  class RadosWriter {
    + process(data, offset) int
  }
  RadosWriter --|> DataProcessor

  class HeadObjectProcessor {
    - unit64_t head_chunk_size 
    - bufferlist head_data
    - DataProcessor* processor
    - uint64_t data_offest
    # process_first_chunk(data, processor) int
  }
  HeadObjectProcessor --|> ObjectProcessor

  class StripeGenerator {
    <<interface>>
    + next(uint64_t offset, uint64_t *stripe_size) int
  }

  class ManifestObjectProcessor {
    # ChunkProcessor chunk
    # StripeProcessor stripe
    # RadosWriter writer;
    # RGWObjManifest manifest;
    # RGWObjManifest::generator manifest_gen;
    # next(uint64_t offset, uint64_t *stripe_size) int
  }
  ManifestObjectProcessor --* RadosWriter
  ManifestObjectProcessor --* ChunkProcessor
  ManifestObjectProcessor --* StripeProcessor

  ManifestObjectProcessor --|> StripeGenerator
  ManifestObjectProcessor --|> HeadObjectProcessor

  class AtomicObjectProcessor {
    - bufferlist first_chunk
    - process_first_chunk(data, processor) int
    + prepare() int
    + complete() int
  }

  AtomicObjectProcessor --|> ManifestObjectProcessor
```


在{% post_link 'RGW Data Layout' %}中我们知道rgw 对象有几个size相关的参数：
1. Head size
2. Stripe size
3. Chunk size


在RGW整体上传流程中，对Head、Stripe、Chunk的处理采用的是责任链模式。


```mermaid
---
title: RGW Put Obj 流程图
---
flowchart LR
HeadObjectProcessor --> StripeProcessor --> ChunkProcessor --> RadosWriter
```

```mermaid
---
title: RGW Put Obj 流程图
---
flowchart TB
    START@{ shape: circ }
    A@{ shape: rect, label: "len = get_data(data)"}
    B@{ shape: rect, label: "HeadObjectProcessor::process(data, ofs)" }
    C@{ shape: diam, label: "len > 0" }
    D@{ shape: rect, label: "ofs += len"}
    E@{ shape: rect, label: "HeadObjectProcessor::process({}, ofs)" }
    START --> A --> B --> C --Y--> D --> A
    C --N--> E --> STOP@{shape: double-circle}
```