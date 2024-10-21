---
uuid: 80eeedf0-8f81-11ef-87b8-b57bfc12ab55
title: rgw-put-obj-atomic
abbrlink: 44522
date: 2024-10-21 15:53:04
updated:
tags: rgw
categories: ceph
---


## 整体上传的对象的结构



```plantuml
@startuml

interface DataProcessor {
}

interface ObjectProcessor extends DataProcessor {}

class HeadObjectProcessor extends ObjectProcessor {}


class RadosWriter extends DataProcessor{}

class ManifestObjectProcessor extends HeadObjectProcessor, StripeGenerator{}

class AtomicObjectProcessor extends ManifestObjectProcessor{}
@enduml
```