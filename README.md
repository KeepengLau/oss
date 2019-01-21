# oss接入文档

## 目录
* [概念](#概念)
















## 概念

- 对象存储服务（Object Storage Service，简称 OSS）
- 存储空间（Bucket）    
  存储空间是用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。   
  可以设置和修改存储空间属性用来控制地域、访问权限、生命周期等，这些属性设置直接作用于该存储空间内所有对象，因此可以通过灵活创建不同的存储空间来完成不同的管理功能。（**注意：目前该功能暂不对外开放**）      
- 对象/文件（Object）   
  对象是 OSS 存储数据的基本单元，也被称为OSS的文件。对象由元信息（Object Meta），用户数据（Data）和文件名（Key）组成。对象由存储空间内部唯一的Key来标识。对象元信息是一个键值对，表示了对象的一些属性，比如最后修改时间、大小等信息，同时也可以在元信息中存储一些自定义的信息。
