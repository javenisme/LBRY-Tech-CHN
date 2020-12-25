# LBRY-Tech-CHN

## 概述

本项目为大家协同LBRY相关技术文档翻译而建，主要文档地址：https://lbry.tech

## 翻译认领

1. https://lbry.tech/spec  已完成部分
2. https://lbry.tech/overview  
3. https://lbry.tech/api/blockchain 待定
4. https://lbry.tech/api/sdk 待定
5. https://lbry.tech/glossary 
6. https://lbry.tech/resources/android-build 
7. https://lbry.tech/resources/wallet-server 
8. https://lbry.tech/resources/daemon-settings
9. https://lbry.tech/contribute?_ga=2.217142302.969493332.1608786133-301747654.1608555029


## 核心术语约定

术语 | 释义
--------- | -------------
blob |  数据网络上的数据传输单位。一个已发布的文件会被分割成许多blob
stream |  一组blob可以重新组合成一个文件。每个流都有一个或多个包含发布文件的内容blob和一个包含内容blob哈希值列表的manifest blob
blob hash |  一个blob的加密哈希值。哈希值用于唯一识别blob，并验证blob的内容是否正确。除非另有规定，LBRY使用SHA-384作为哈希函数
metadata |  流的内容信息（如创建者、描述、流哈希等），元数据存储在区块链中
name | 与Claim相关联的人可读的UTF8字符串
stake | 区块链中的一个条目，预留了一些信用，并将其与名称关联起来
claim | 包含了关于流或频道的元数据Stake
support | 借贷Credits（译者注：通证或代币）来奖励Claim的Stake
channel | 匿名发布者身份的标识。Claims可以是频道的一部分
URL | A memorable reference to a claim.

## 勘误

## 鸣谢贡献

## 参与讨论
欢迎加入Discord [LBRY中文社区](https://discord.gg/YxT8Yz7e) 




