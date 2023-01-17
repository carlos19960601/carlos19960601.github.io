---
title: "RN开发笔记"
date: 2023-01-17T22:06:47+08:00
draft: true
original: true
categories: 
  - ReactNative
tags: 
  - ReactNative
---

本文主要是记录我在学习RN过程中遇到的问题，以及解决方式。

# 开发相关

<!--more-->

# 打包相关

1. 根据[官网文档](https://reactnative.dev/docs/signed-apk-android)使用`./gradlew bundleRelease`构建出来的是aab格式的安装包，不能直接安装，如果要打包成apk文件需要使用命令`./gradlew assembleRelease`

2. 打包成apk文件后，安装到手机上后，发现没有网络请求，由于没有域名也没主机，所以代码中后端地址配置的是`http://局域网ip`;原因是android9.0后默认禁止访问不安全的请求，比如http。所以解决方法是
   1. 使用https
   2. 配置成可以水用http
      1. 在res下新增加一个xml目录，然后创建一个名为network_security_config.xml文件, 文件内容如下
      ```
      <?xml version="1.0" encoding="utf-8"?>
      <network-security-config>
          <base-config cleartextTrafficPermitted="true" />
      </network-security-config>
      ```
      2. 在androidManifiest.xml文件中添加
      ```
       android:networkSecurityConfig="@xml/network_security_config"
      ```