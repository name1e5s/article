---
title: SGX 入门 —— 基础知识介绍
date: 2019-11-10 00:10:50
tags: 
    - Intel SGX
---

### Intel SGX 是什么

英特尔® 软件防护扩展 SGX (Software Guard Extensions) 是一项面向应用程序开发人员的英特尔技术｡ 其通过在应用程序的地址空间内开辟的被称为 **Enclave** (直译为飞地) 的容器，为需要安全保证的的代码和数据提供机密性和完整性的保护,使之免受拥有特殊权限的恶意软件的破坏。这一技术由最近几代 Intel CPU 内置的 Memory Encryption Engine 提供，整个 SGX 的概念图如下：

![[Source](https://qiita.com/Cliffford/items/2f155f40a1c3eec288cf)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F349383%2Fb94bbf4b-f1bc-2a13-a08d-87bd64391dbf.png?ixlib=rb-1.2.2&auto=compress%2Cformat&gif-q=60&s=ddfd8975727c71c25757cfada3c8427c)

SGX 的作用需要处理器提供指令支持，因此其对硬件的版本有要求。按照官方的说法，只有在第六代（Skylake）后的设备才可以启用 Intel SGX。

### Intel SGX 可以应对的攻击

![Source: Graphene-SGX: A Practical Library OS for Unmodified Applications on SGX](https://raw.githubusercontent.com/name1e5s/article/master/pic/SGX.png)


