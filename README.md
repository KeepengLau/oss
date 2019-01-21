# oss接入文档

## 目录
* [概念](#概念)
* [编程模型](#编程模型)
* [基本参数](#基本参数)
* [安全机制](#安全机制)
* [上传凭证](#上传凭证)












## 概念

- 对象存储服务（Object Storage Service，简称 OSS）
- 存储空间（Bucket）    
  存储空间是用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。   
  可以设置和修改存储空间属性用来控制地域、访问权限、生命周期等，这些属性设置直接作用于该存储空间内所有对象，因此可以通过灵活创建不同的存储空间来完成不同的管理功能。（**注意：目前该功能暂不对外开放**）      
- 对象/文件（Object）   
  对象是 OSS 存储数据的基本单元，也被称为OSS的文件。对象由元信息（Object Meta），用户数据（Data）和文件名（Key）组成。对象由存储空间内部唯一的Key来标识。对象元信息是一个键值对，表示了对象的一些属性，比如最后修改时间、大小等信息，同时也可以在元信息中存储一些自定义的信息。
## 编程模型
![](https://i.imgur.com/q6CVwWE.png)
## 基本参数
- **AccessKeyId(测试)：** pr4*************fg
- **AccessKeySecret(测试)：** Kgt***************Krn3
- **请求地址（测试环境）：** http://oss.test.etcsd.cn/
- **请求地址（生产环境）：** https://oss.etcsd.com/
每个产品组可以根据项目申请AccessKey(AccessKeyId+AccessKeySecret),每个AccessKey默认有自己独立的存储空间（Bucket），各组可根据实际情况进行申请。后续将根据情况开发存储空间的管理接口。
## 安全机制

   OSS通过使用AccessKeyId/AccessKeySecret对称加密的方法来验证某个请求的发送者身份。AccessKeyId用于标示用户，AccessKeySecret是用户用于加密签名字符串和OSS用来验证签名字符串的密钥，其中AccessKeySecret必须保密

## 上传凭证
   
1. 构造上传策略，目前仅支持deadline这一个策略，后续根据实际需求可扩展
   
    字段名 | 类型 | 必填 | 说明
     ---| ---| --- | ---
    **deadline** | int | 是 | 上传凭证有效截止时间。Unix时间戳，单位为秒。一般建议设置为上传开始时间 + 3600s，用户可根据具体的业务场景对凭证截止时间进行调整。
    **is_public_access** | int | 否 | 0：私有资源(默认)； 1：公开资源
    **is_encrypted_storage** | int | 否 | 0：默认； 1：加密存储


2. 将上传策略序列化为json，如下所示
    >  {"deadline":1544599494}

    或

    > {"deadline":1544599494, "is_public_access":1}
    

3. 对 JSON 编码的上传策略进行[URL 安全的 Base64 编码](#1)，得到待签名字符串：
    > policy = {"deadline":1544599494}
    > 
    > encodedPolicy = urlsafe_base64_encode(policy)
    > 
    > 计算结果为：eyJkZWFkbGluZSI6MTU0NDU5OTQ5NH0=
    
4. 使用AccessKeySecret对上一步生成的待签名字符串计算HMAC-SHA1签名：
    > sign = hmac_sha1(encodedPolicy, AccessKeySecret)
    > 
    > 签名结果为: ccf06e475646d8bea8f6d4d07fe8fbd4d842dfd8
    >
   **注意：**签名结果是二进制数据，此处输出的是每个字节的十六进制表示，以便核对检查。

5. 对签名进行[URL 安全的 Base64 编码](#1)：
    > encodedSign = urlsafe_base64_encode(sign)
    > 
    > 最终签名值为: Y2NmMDZlNDc1NjQ2ZDhiZWE4ZjZkNGQwN2ZlOGZiZDRkODQyZGZkOA==
    >
6. 将**AccessKeyId**、**encodedSign** 和 **encodedPolicy** 用英文符号:连接起来：
    > token = AccessKeyId + ":" + encodedSign + ":" + encodedPolicy
    > 
    > 实际值为: pr**********kfg:Y2NmMDZlNDc1NjQ2ZDhiZWE4ZjZkNGQwN2ZlOGZiZDRkODQyZGZkOA==:eyJkZWFkbGluZSI6MTU0NDU5OTQ5NH0=
7.  将token填充在上传资源请求Header Authorization 域中，格式为：
  
    >**UpToken** \<token> 
    >
    **注意中间有空格**
