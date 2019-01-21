# oss接入文档

## 目录
* [概念](#概念)
* [编程模型](#编程模型)
* [基本参数](#基本参数)
* [安全机制](#安全机制)
* [上传凭证](#上传凭证)
* [下载凭证](#下载凭证)
* [API参考](#api参考)
  * [公共HTTP头定义](#公共http头定义)
  * [上传文件](#上传文件)
  * [上传Base64格式文件](#上传base64格式文件)
  * [下载资源](#下载资源)
  * [错误响应](#错误响应)
* [数据格式](#数据格式)
  * [URL安全的Base64编码](#url安全的base64编码)









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
7.  将token填充在上传资源请求Header Authorization 域中，格式为(不包含尖括号)：
  
    >**UpToken** \<token> 
    >
    **注意中间有空格**
## 下载凭证

1. 下载URL,key为上传时服务返回:
    > http://oss.test.etcsd.cn/object/{key} 如:         
    > http://oss.test.etcsd.cn/object/5c10cf2a43b8e4403afc25e4
   
2. 为下载 URL 加上过期时间 e 参数，Unix时间戳：
    > DownloadUrl = 'http://oss.test.etcsd.cn/object/5c10cf2a43b8e4403afc25e4?e=1544778173'
   
3. 对上一步得到的 URL 字符串计算HMAC-SHA1签名，并对结果做**URL安全的 Base64 编码**：
    > Sign = hmac_sha1(DownloadUrl, AccessKeySecret)
    > 
    > EncodedSign = urlsafe_base64_encode(Sign)

4. 将AccessKeyId与上一步计算得到的结果用英文符号 : 连接起来：
    > token = pr43dzwwhpfh9w4d7kfg:NTFlMDJkNTMyZGVmZjMyZmRkMjIwNGY1NzEwODUyOTcwNTdhY2IwNQ==

5. 将上述Token 拼接到含过期时间参数 e 的 DownloadUrl 之后，作为最后的下载 URL,通过get方法调用即可：
    > RealDownloadUrl = http://oss.test.etcsd.cn/object/5c10cf2a43b8e4403afc25e4?e=1544778173&token=pr43dzwwhpfh9w4d7kfg:NTFlMDJkNTMyZGVmZjMyZmRkMjIwNGY1NzEwODUyOTcwNTdhY2IwNQ==

    **RealDownloadUrl** 即为下载对应私有资源的可用 URL ，并在指定时间后失效。失效后，可按需要重新生成下载凭证。
## API参考
### 公共HTTP头定义


Http header |  描述  | 类型 
--- | --- |--- |
Authorization | 用于验证请求合法性的认证信息。该头部应严格按照上传凭证格式进行填充，否则会返回 401 错误码。例如Authorization: UpToken pr43dzwwhpfh9w4d7kfg:NjZmYzFmYzA... | 请求头
md5 | 上传内容的 md5 校验码。如果指定此值，则OSS服务器会使用此值进行内容检验 | 请求头
X-OSS-Request-Id | 上传请求的唯一 ID。通过该 ID 可快速定位用户请求的相关日志 | 响应头


### 上传文件

目前仅支持表单上传，即在一个单一的 HTTP POST 请求中完成一个文件的上传，比较适合简单的应用场景和尺寸较小的文件。

#### 请求
    /object/upload
#### 语法
请求报文的内容以**multipart/form-data**格式组织，上传凭证放在Http请求头(Header)中：
> 
> **POST** /object/upload  HTTP/1.1
> 
> **Host**:   \<UpHost>
> 
> **Content-Type**:   multipart/form-data; boundary=\<frontier>
> 
> **Content-Length**: \<multipartContentLength>
> 
> **Authorization**: UpToken \<UploadToken>
> 
> --\<frontier>
> 
> Content-Disposition:   form-data; name="**file**"; filename="\<fileName>"
> 
> Content-Type:  application/octet-stream
> 
> Content-Transfer-Encoding: binary
> 
> \<fileBinaryData>
> 
> --\<frontier>--


#### 表单域
名称 | 类型 | 描述 | 必填
---| --- | --- | ---
**file** | 字符串 | 文件或文本内容，必须是表单中的最后一个域。浏览器会自动根据文件类型来设置Content-Type，会覆盖用户的设置。 OSS一次只能上传一个文件。默认值：无 | 是


#### 请求Header

名称 | 类型 | 描述
--- | --- | ---
**Authorization** | 字符串 | 用于验证请求合法性的认证信息。该头部应严格按照上传凭证格式进行填充，否则会返回 401 错误码。例Authorization: UpToken 3vbzhbbdkvb9rv7zhqcx:NjZmYzFmYzA2OGYxNWQ...。

#### 响应
响应成功返回Http状态码：**200**
#### 响应报文
**json格式：**

名称 | 类型 | 说明
--- | --- | ---
**md5** | 字符串 | 目标资源的MD5值
**key** | 字符串 | 目标资源的唯一标识，由OSS服务器生成

### 上传Base64格式文件
#### 请求
    /object/upload/encoded
#### 语法
请求报文的内容以**x-www-form-urlencoded**格式组织，上传凭证放在Http请求头(Header)中：
> 
> **POST** /object/upload/encoded  HTTP/1.1
> 
> **Host**:   \<UpHost>
> 
> **Content-Type**:   application/x-www-form-urlencoded;charset=utf-8
> 
> 
> **Authorization**: UpToken \<UploadToken>
> 
> **filename=test.jpg&binary=/9j/4AAQSkZJRg...**

字段名称 | 类型 | 说明
---|---|---
filename|string|文件原始名字。如test.jpg
binary|string|base64编码之后的数据。 Base64Encode(test.jpg)
### 下载资源
#### 请求
    /object/<key>
#### 公共资源下载
公开资源下载通过 HTTP GET 的方式访问资源 URL 即可。资源 URL 的构成如下：

    http://<server>/object/<key>
#### 私有资源下载
私有资源下载是通过HTTP GET的方式访问特定的 URL。私有资源URL与公开资源URL相比只是增加了两个参数e和token，分别表示过期时间和下载凭证。一个完整的私有资源 URL 如下所示：

    http://<server>/object/<key>?e=<deadline>&token=<downloadToken>
#### 响应
响应成功返回Http状态码：200
#### 响应报文
无

## 错误响应

### 错误响应格式
当请求出现错误时，响应头部信息包括：


- Content-Type: application/json


- 一个合适的 3xx，4xx，或者 5xx 的 HTTP 状态码
各个接口在遇到执行错误时，将返回一个 JSON 格式组织的信息对象，描述出错原因。具体格式如下：
```
{   
     "code": \<int http code>,    
     "message": \<string errmsg>"   
}
```


字段名称|类型|说明
---|---|---
**code**| int | 返回的错误码，用来定位错误场景。
**message**| String | 包含详细的错误信息

## URL安全的Base64编码

URL安全的Base64编码适用于以URL方式传递Base64编码结果的场景。该编码方式的基本过程是先将内容以Base64格式编码为字符串，然后检查该结果字符串，将字符串中的加号+换成中划线-，并且将斜杠/换成下划线_。
