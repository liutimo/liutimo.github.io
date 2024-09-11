---
uuid: 3803c071-5f7f-11ef-a32b-fbc9b553a498
title: rgw对象上传1
date: 2024-08-01 10:59:57
updated:
tags:
categories:
---

> rgw支持S3和Swift协议，我们目前主要分析S3协议下的对象上传流程。

```plantuml
@startuml

class RGWOp

class RGWPutObj extends RGWOp

class RGWPutObj_ObjStore extends RGWPutObj {
  + verify_params() : int
  + get_params() : int
  + get_data(bufferlist& bl) : int
}

class RGWPutObj_ObjStore_S3 extends RGWPutObj_ObjStore {
  + get_params() : int
  + get_data(bufferlist& bl) : int
}

class RGWPutObj_ObjStore_SWIFT extends RGWPutObj_ObjStore

@enduml
```


```c++
RGWOp::execute
  sal::Bucket::check_quota

```


```plantuml
@startuml
class StripeGenerator  {

}

class DataProcessor {
  + process(data, offset) : int
}

class ObjectProcessor extends DataProcessor {

}


class HeadObjectProcessor extends ObjectProcessor {

}

class ManifestObjectProcessor extends HeadObjectProcessor, StripeGenerator {
  - chunk : ChunkProcessor
  - stripe : StripeProcessor
  - writer : RadosWriter
  - head_obj : unique_ptr<sal::Object>
}

ManifestObjectProcessor --> ChunkProcessor : uses
ManifestObjectProcessor --> StripeProcessor : uses
ManifestObjectProcessor --> RadosWriter : uses

class AtomicObjectProcessor extends ManifestObjectProcessor {
  - process_first_chunk(data, processor) : int
}


class MultipartObjectProcessor extends ManifestObjectProcessor {
  
}

class AppendObjectProcessor extends ManifestObjectProcessor {
  
}

class RadosWriter extends DataProcessor {
  - head_obj : unique_ptr<sal::Object>
  - stripe_obj : RGWSI_RADOS::Obj
}

class Pipe extends DataProcessor {
  
}

class ChunkProcessor extends Pipe {

}

class StripeProcessor extends Pipe {

}

class Writer extends ObjectProcessor {

}

class RadosAtomicWriter extends Writer {
  - processor : AtomicObjectProcessor

  + prepare() : int
  + process(data, offset) : int
  + complete(...) : int
}
RadosAtomicWriter --> AtomicObjectProcessor : uses

class RadosAppendWriter extends Writer {

}

class RadosMultipartWriter extends Writer {

}

@enduml
```

`RadosAtomicWriter`本质上就是`AtomicObjectProcessor`的 wrapper/proxy。

`AtomicObjectProcessor::prepare`计算出`head size` 和 `stripe size`,并初始化`ChunkProcessor`和`StripeProcessor`



目前，rgw支持三种上传方式：
1. multipart
2. append
3. atomic


## 通用流程

```plantuml
@startuml
start
:get_xxx_writer;
note left
atomic: RadosAtomicWriter
append: RadosAppendWriter
multipart: RadosMultipartWriter
end note
:ObjectProcessor::prepare;
:ObjectProcessor::process;
:ObjectProcessor::complete;
end
@enduml
```

## Atomic上传
### RadosAtomicWriter
```plantuml
@startuml
class DataProcessor {
  # virtual int process(bufferlist&&, uint64_t) = 0;
}

class ObjectProcessor extends DataProcessor {
  + virtual int prepare(...) = 0;
  + virtual int complete(...) = 0;
}

class Writer extends ObjectProcessor {
  //继承父类所有的纯虚函数，没有实现。
}

class RadosAtomicWriter extends Writer {
  # store : rgw::sal::RadosStore;
  # aio: std::unique_ptr<Aio>;
  # processor: AtomicObjectProcessor;
}
@enduml
```

接下来看看`RadosAtomicWriter`的实现：
![alt text](image.png)

所以，`RadosAtomicWriter`其实就是`AtomicObjectProcessor`的proxy。rgw为啥这样设计呢？
{% note warning %}
[**待修改**]在旧版本的rgw代码中，Writer相关的API和Rados的耦合十分严重。从Qunicy版本，rgw引入了DBStore，为了兼容不同的存储底座，需要抽象出一套通用的API（其实就是Writer，从Store::get_xxx_writer的返回值就可以看出来）用于对象上传。
同时，为了复用已有的Rados Atomic上传的代码，就使用了代理模式，使之兼容新的Writer API。
[其实我也搞不清这是代理模式还是适配器模式](https://www.cnblogs.com/gocode/p/proxy-pattern-and-adapter-pattern.html)
{% endnote %}

### AtomicObjectProcessor

```plantuml
@startuml
class HeadObjectProcessor extends ObjectProcessor {
  - head_chunk_size: uint64_t
  - head_data: bufferlist
  - processor: DataProcessor
  - data_offset: uint64_t
  # virtual int process_first_chunk(bufferlist&&, DataProcessor **) = 0;
  + int process(bufferlist&& data, uint64_t logical_offset) final override;
}

class StripeGenerator {
  + virtual int next(uint64_t offset, uint64_t *stripe_size) = 0;
}

class ManifestObjectProcessor extends HeadObjectProcessor, StripeGenerator {

}

class AtomicObjectProcessor extends ManifestObjectProcessor {
  - int process_first_chunk(bufferlist&&, rgw::sal::DataProcessor**) override;
}
@enduml
```


```plantuml
@startuml
start
:AtomicObjectProcessor::prepare;
end
@enduml
```


```plantuml
@startuml
class rgw_placement_rule {
  + name: std::string
  + storage_class: std::string
}
@enduml
```