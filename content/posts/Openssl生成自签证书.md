---
title: "Openssl生成自签证书"
date: 2022-05-06T19:53:54+08:00
draft: false
original: false
categories: 
  - Tools
tags: 
  - OpenSSL
---

**创建配置文件**

openssl.conf:
```
# ca根证书配置
[ ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true

# HTTPS应用证书配置
[ crt ]
subjectKeyIdentifier = hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical, CA:false
keyUsage = critical, digitalSignature, cRLSign, keyEncipherment
extendedKeyUsage = critical, serverAuth, clientAuth
subjectAltName=@alt_names

# SANs可以将一个证书给多个域名或IP使用
# 访问的域名或IP必须包含在此，否则无效
# 修改为你要保护的域名或者IP地址，支持通配符
[alt_names]
DNS.1 = *.dmtech.com
```

<!--more-->

**生成根证书**

1. 生成根证书私钥

```
openssl genrsa -out root.key 2048
```

2. 生成根证书请求文件

> C：所在国家 (Country)，只能是两位字母缩写
> 
> ST：所在省份(State)
> 
> L：所在城市(Locality)
> 
> O：所在组织(Organization)
> 
> OU：所在部门(Organization Unit)
> 
> CN：公用名，即域名(Common Name)

```
openssl req -new -key root.key  -out root.csr -subj "/C=CN/ST=Sichuan/L=Chengdu/O=dmtech/OU=dmtech/CN=dmtech.com"
```

3. 生成根证书

> -req: 证书请求
> 
> -extfile：扩展文件配置和-extensions参数使用
> 
> -extensions：扩展配置
> 
> -in：证书请求文件
> 
> -out：证书输出文件
> 
> -signkey：签名的私钥
> 
> -days：有效期

```
openssl x509 -req -extfile openssl.cnf -extensions ca -in root.csr -out root.crt -signkey root.key -CAcreateserial -days 3650
```

**签发应用证书**

1. 生成应用私钥

```
openssl genrsa -out harbor.key 2048
```

2. 生成harbor.dmtech.com域名证书请求文件

```
openssl req -new -key harbor.key -out harbor.csr -subj "/C=CN/ST=Sichuan/L=Chengdu/O=dmtech/OU=dmtech/CN=harbor.dmtech.com"
```

3. 签发证书

```
openssl x509 -req -extfile openssl.cnf -extensions crt -CA root.crt -CAkey root.key -CAserial harbor.srl -CAcreateserial -in harbor.csr -out harbor.crt -days 3650
```

4. 使用应用私钥harbor.key和应用证书harbor.crt为harbor.dmtech.com添加HTTPS服务