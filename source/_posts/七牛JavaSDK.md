---
title: 七牛JavaSDK
comments: true
date: 2017-07-10 10:58:30
tags:
- 七牛
categories:
- 技术
---


### 介绍

[官方文档](https://developer.qiniu.com/kodo/sdk/1239/java)

Java SDK 属于七牛服务端SDK之一，主要有如下功能：

1. 提供生成客户端上传所需的上传凭证的功能
2. 提供文件从服务端直接上传七牛的功能
3. 提供对七牛空间中文件进行管理的功能
4. 提供对七牛空间中文件进行处理的功能
5. 提供七牛CDN相关的刷新，预取，日志功能

### 安装

**Gradle**

compile 'com.qiniu:qiniu-java-sdk:7.2.+'

**Maven**

<dependency>
  <groupId>com.qiniu</groupId>
  <artifactId>qiniu-java-sdk</artifactId>
  <version>[7.2.0, 7.2.99]</version>
</dependency>


### 鉴权

七牛账号需要通过一对密钥访问, 分别是`Access Key`和`Secret Key`, 可在网站的**个人中心->密钥管理**中查看

### 文件上传

上传需要几个基本参数：`AccessKey`, `SecretKey`, `bucket`, `Key`

`AccessKey`: 个人中心->密钥管理中获取
`SecretKey`: 个人中心->密钥管理中获取
`bucket`: 七牛中的存储空间名称
`Key`: 图片文件名称

请求返回值，包含`Key`和`Hash`，如果不带key值，则两个值一样, 请求的时候用Url+Key访问。

#### 客户端上传

APP端、Web端如果要上传图片，需要向服务端请求token， 请求方式如下：

```java
String accessKey = "access key";
String secretKey = "secret key";
String bucket = "bucket name";

Auth auth = Auth.create(accessKey, secretKey);
String upToken = auth.uploadToken(bucket);
System.out.println(upToken);

```

客户端拿到`token`之后，通过`post`请求`http://upload.qiniu.com/`，上传文件和`token`参数即可。

#### 服务端上传

##### 文件上传

```java
//构造一个带指定Zone对象的配置类
Configuration cfg = new Configuration(Zone.zone0());
//...其他参数参考类注释

UploadManager uploadManager = new UploadManager(cfg);
//...生成上传凭证，然后准备上传
String accessKey = "your access key";
String secretKey = "your secret key";
String bucket = "your bucket name";
//如果是Windows情况下，格式是 D:\\qiniu\\test.png
String localFilePath = "/home/qiniu/test.png";
//默认不指定key的情况下，以文件内容的hash值作为文件名
String key = null;

Auth auth = Auth.create(accessKey, secretKey);
String upToken = auth.uploadToken(bucket);

try {
    Response response = uploadManager.put(localFilePath, key, upToken);
    //解析上传成功的结果
    DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
    System.out.println(putRet.key);
    System.out.println(putRet.hash);
} catch (QiniuException ex) {
    Response r = ex.response;
    System.err.println(r.toString());
    try {
        System.err.println(r.bodyString());
    } catch (QiniuException ex2) {
        //ignore
    }
}

```

##### 字节数组上传

```java
//构造一个带指定Zone对象的配置类
Configuration cfg = new Configuration(Zone.zone0());
//...其他参数参考类注释

UploadManager uploadManager = new UploadManager(cfg);
//...生成上传凭证，然后准备上传
String accessKey = "your access key";
String secretKey = "your secret key";
String bucket = "your bucket name";

//默认不指定key的情况下，以文件内容的hash值作为文件名
String key = null;

try {
    byte[] uploadBytes = "hello qiniu cloud".getBytes("utf-8");
    Auth auth = Auth.create(accessKey, secretKey);
    String upToken = auth.uploadToken(bucket);

    try {
        Response response = uploadManager.put(uploadBytes, key, upToken);
        //解析上传成功的结果
        DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
        System.out.println(putRet.key);
        System.out.println(putRet.hash);
    } catch (QiniuException ex) {
        Response r = ex.response;
        System.err.println(r.toString());
        try {
            System.err.println(r.bodyString());
        } catch (QiniuException ex2) {
            //ignore
        }
    }
} catch (UnsupportedEncodingException ex) {
    //ignore
}

```

##### 数据流上传

```java
//构造一个带指定Zone对象的配置类
Configuration cfg = new Configuration(Zone.zone0());
//...其他参数参考类注释

UploadManager uploadManager = new UploadManager(cfg);
//...生成上传凭证，然后准备上传
String accessKey = "your access key";
String secretKey = "your secret key";
String bucket = "your bucket name";

//默认不指定key的情况下，以文件内容的hash值作为文件名
String key = null;

try {
    byte[] uploadBytes = "hello qiniu cloud".getBytes("utf-8");
    ByteArrayInputStream byteInputStream=new ByteArrayInputStream(uploadBytes);
    Auth auth = Auth.create(accessKey, secretKey);
    String upToken = auth.uploadToken(bucket);

    try {
        Response response = uploadManager.put(byteInputStream,key,upToken,null, null);
        //解析上传成功的结果
        DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
        System.out.println(putRet.key);
        System.out.println(putRet.hash);
    } catch (QiniuException ex) {
        Response r = ex.response;
        System.err.println(r.toString());
        try {
            System.err.println(r.bodyString());
        } catch (QiniuException ex2) {
            //ignore
        }
    }
} catch (UnsupportedEncodingException ex) {
    //ignore
}

```

##### 断电续传

```java
//构造一个带指定Zone对象的配置类
Configuration cfg = new Configuration(Zone.zone0());
//...其他参数参考类注释

//...生成上传凭证，然后准备上传
String accessKey = "your access key";
String secretKey = "your secret key";
String bucket = "your bucket name";
//如果是Windows情况下，格式是 D:\\qiniu\\test.png
String localFilePath = "/home/qiniu/test.mp4";
//默认不指定key的情况下，以文件内容的hash值作为文件名
String key = null;

Auth auth = Auth.create(accessKey, secretKey);
String upToken = auth.uploadToken(bucket);

String localTempDir = Paths.get(System.getenv("java.io.tmpdir"), bucket).toString();
try {
    //设置断点续传文件进度保存目录
    FileRecorder fileRecorder = new FileRecorder(localTempDir);
    UploadManager uploadManager = new UploadManager(cfg, fileRecorder);
    try {
        Response response = uploadManager.put(localFilePath, key, upToken);
        //解析上传成功的结果
        DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
        System.out.println(putRet.key);
        System.out.println(putRet.hash);
    } catch (QiniuException ex) {
        Response r = ex.response;
        System.err.println(r.toString());
        try {
            System.err.println(r.bodyString());
        } catch (QiniuException ex2) {
            //ignore
        }
    }
} catch (IOException ex) {
    ex.printStackTrace();
}

```