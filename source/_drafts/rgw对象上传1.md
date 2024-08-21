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
1. 分段上传
2. 原子上传
3. 追加写


## 原子上传

get_data： 4M HEAD

```plantuml
@startuml
class HeadObjectProcessor {
  # virtual int process_first_chunk(data, processor<out>)
}
@enduml
```

```plantuml
@startuml
class HeadObjectProcessor {
  # virtual int process_first_chunk(data, processor<out>)
}
@enduml
```