---
title: spring security+jwt鉴权授权
date: 2023-02-24 11:32:00
categories:
- Java
tags:
- springboot 
- jwt
- security
---

### JWT(JSON Web TOken)

jwt由以下三部分组成

* header

header用于存放签名使用的算法

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

* payload

payload存放用户的信息、token的生成时间、过期时间等有效信息

```json
{
    "sub":"用户名",
    "created":"token生成时间",
    "exp":"token过期时间"
}
```

* signature

signature是一个签证信息，由header、payload、secret组成。

jwt的签发和生成在服务端，secret用来jwt的签发生成和认证（保存在服务端），相当于服务端的私钥。

### 认证流程

jwt一般用于客户端和服务端之间认证用户的身份信息，用户身份认证通过后，才允许用户访问服务端资源。

1. 客户端携带凭证（账号密码）请求服务端认证，服务端验证凭证信息，认证通过签发jwt并返回给客户端，客户端存储jwt。
2. 客户端携带jwt请求服务端资源，服务端校验jwt合法性（是否正确、是否过期等），合法则将服务端资源返回给客户端。

### java使用jwt

#### 导入依赖

```xml
```

