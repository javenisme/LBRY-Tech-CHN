#LBRY 一个去中心化的数字内容交易市场

原文：LBRY: A Decentralized Digital Content Marketplace
地址：https://lbry.tech/spec
作者：Alex Grintsvayg, Jeremy Kauffman
译者：Javen Yeung 于2020年平安夜

## 目录

[TOC]


## 简介
LBRY是一个在全球去中心化市场中访问和发布数字内容的协议。LBRY使用公共区块链来提供单一的已发布内容的共享索引，以及内容发现和支付。

客户端可以使用LBRY来发布、托管、查找、下载和为众多内容付费--书籍、电影、音乐或其他任何可以表示为比特流的内容。该协议是无需授权和抗审查的，这意味着每个人都可以参与，没有人可以单方面阻止或删除内容。

在LBRY之前，出版商不得不在亚马逊或Youtube这样的中心化平台，或者Bittorrent这样的协议之间做出选择。中心化平台存在几个问题，因为它们的激励机制与用户的激励机制不一致。平台从事寻租行为，经常抽取创作者利润的30-55%。他们对创作者实施不透明和任意的规则，并在没有警告或社区意见的情况下改变这些规则。他们在世界各地的极权统治的要求下选择审查内容，以换取更多的用户和更高的利润。

Bittorrent没有这些缺点，但它有自己的问题。只有当人们已经知道他们所寻找的内容的信息哈希值时，它才会有用，而在协议中没有办法发现这些哈希值。即使使用外部搜索引擎也不能提供网络上可用的全面列表。没有激励用户为内容作种的措施，Bittorrent的工作很大程度上是因为用户通过私人社区获得关注，作种者根本不了解他们的客户在做什么。最后，BitTorrent上的许多内容侵犯了版权，这损害了协议的公众看法，掩盖了它的许多积极因素。

LBRY比这两种选择(译者注：中心化平台和BitTorrent)都有重大改进。它使用区块链来提供中心化平台的好的部分（一个存储数据、寻找有趣内容、建立品牌和获得贡献奖励的单一地方），同时消除了缺点（不透明、任意规则、租金提取和审查制度）。它是公开的，任何人都不能被审查或阻止使用它。它的规则是明确的，在没有社区共识的情况下无法更改。区块链记录了所有发布到LBRY的内容，所以有趣的内容很容易找到，侵权的内容很难隐藏。访问区块链数据是免费的，下载内容的成本是透明的，发布者按其设定的价格100%赚取。

### 进度
LBRY自2016年6月开始公开使用。截至2020年5月，已经有超过330万条数字内容通过该协议发布。每月有数百万用户访问这些内容，下载和上传TB级数据。图形浏览器和钱包适用于所有主流操作系统，可以在lbry.com/get下载。

### 概述
本文档定义了LBRY协议、其组件以及它们是如何结合在一起的。LBRY由几个离散的组件组成，这些组件一起使用，以提供协议的端到端功能。有两个分布式数据存储（区块链和DHT），一个用于交换数据的点对点协议，以及数据结构、编码和检索的规范。

### 前提假设
本文档假设读者熟悉分布式哈希表（DHTs）、BitTorrent协议、比特币和一般的区块链技术。它并不试图记录这些技术或解释它们如何工作。建议希望了解技术细节的人阅读比特币开发者参考资料和BitTorrent协议规范。

### 公约和术语

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
URL | A memorable reference to a claim

## Blockchain
LBRY区块链是一个公开的工作证明区块链。其设计基于比特币，但做了大量修改。本文档不涉及或具体述说LBRY与比特币相同的任何方面，而是专注于不同之处，主要是Claim操作和claimtrie。

我们的区块链有三个主要目的。

1. 一个网络内容的索引
2. 价格内容的支付系统和购买记录。
3. 加密出版商身份的来源。

### Stakes
Stakes是区块链中的一个单一条目，它用于向一个名字声明凭证（Commits credits）。两种类型的Stakes是Claims和Supports。

所有的Stake都有这些属性：

id  一个20字节的哈希值，在所有Stake中是唯一的。请参阅Stake标识符的生成。
amount  用来支持Stake的代币数量。见控制。

####Claims
Claims是存储元数据的Stake。有两种类型的Claims。流Claims声明流的可用性、访问方法和发布者。频道Claims创建了一个假名，可以作为流Claims的发布者。

##### Claims属性
除了所有股权具有的属性外，Claims还具有两个属性。

name  最多 255 个字节的 UTF-8 字符串，用于处理Claims。参见URLs。
value  流或通道的元数据。参见元数据。

##### Claim例子

下面是一个流Claims的例子：


```
{
  "claimID": "6e56325c5351ceda2dd0795a30e864492910ccbf",
  "amount": 1.0,
  "name": "lbry",
  "value": {
    "stream": {
      "title": "What is LBRY?",
      "author": "Samuel Bryan",
      "description": "What is LBRY? An introduction with Alex Tabarrok",
      "language": "en",
      "license": "Public Domain",
      "thumbnail": "https://s3.amazonaws.com/files.lbry.io/logo.png",
      "mediaType": "video/mp4",
      "streamHash": "232068af6d51325c4821ac897d13d7837265812164021ec832cb7f18b9caf6c77c23016b31bac9747e7d5d9be7f4b752",
    },
  }
}

```

注意：区块链将value视为一个不透明的字节字符串，并不对其施加任何结构。结构会在堆栈的更高处被应用和验证。这里显示的value仅用于演示目的。

##### Claim操作
Claim操作有三种：创建、更新、放弃

create   创建一个Claim
update   改变现有Claim的价值、金额或频道，不改变Claim的ID.
abandon   撤回一项Claim，使相关的贷方得以用于其他目的。


#### Supports
Support是借以amount来激励现有Claim的Stake。

#####Support属性
除了Stake属性外，Support还有一个额外的属性：

ClaimID   该Support正在加强的ID

##### Support的例子
下面是一个支持上述说法的例子：

```
{
  "supportID": "fbcc019294468e03a5970dd2adec1535c52365e6",
  "amount": 45.12,
  "claimID": "6e56325c5351ceda2dd0795a30e864492910ccbf",
}

```


##### Support操作
Support的创建和放弃与Claim一样（见Claim操作）。Claim不能更新，也不能自己Support。

#### Claimtrie
Claimtrie是一个数据结构，用于存储所有Claim的集合，并证明URL解析的正确性。

Claimtrie是以Merkle树的形式来实现的，它将名称映射到Claim上。claims被存储为树中的叶子节点。名字存储为从根节点到叶节点的归一化路径。

根哈希是根节点的哈希值。它存储在区块链中每个区块的头中。节点使用根哈希来高效、安全地验证Claimtrie的状态。

同一名称可以存在多个Claim。它们都存储在该名称的叶子节点中。请参阅索赔排序

关于具体的claimtrie实现，请看源码。

#### Statuses
在一个给定的区块中，Stakes可以有以下一个或多个状态。

#####Accepted

被接受的stake是指已经进入区块链的stake。当包含它的交易被包含在一个区块中时，就会发生这种情况。
被接受的stake在其活跃之前不会影响叶内claim顺序。
一个Claim的金额和它的所有接受的Support的总和被称为它的总金额。

#####Abandoned
被放弃的stake是指被其所有者撤回的stake。花费交易中包含一个stake，将导致该stake被放弃。被放弃的stake会从claimtrie中移除。

虽然与被遗弃的stake相关的数据仍然驻留在区块链中，但它被认为是无效的，不应用于解析URL或获取相关内容。由被放弃的身份签署的活跃的Claim标识符也被认为是无效的。


#####Active
活跃的stake是指在区块链中已被接受且未被放弃的stake，其数量由算法确定。这个所需的时间长度称为激活延迟。

如果该stake是对一个活跃的claim的更新，是一个名称的唯一被接受的非放弃的Claim，或者没有引起控制该名称的Claim的变化，则激活延迟为0（即该股权立即变得活跃）。

否则，激活延迟由激活延迟中的公式决定。该公式的输入是当前区块的高度、接受stake的高度和该名称的控制权最后变化的高度。

一个有效Claim的金额和它所有的有效支持的金额之和称为它的有效金额。有效量会影响叶子节点中的Claim的排序顺序，以及哪个Claim在被该名称（name）控制。不活跃的Claim的有效金额为0。

#####Controlling (Claims仅有)
控制性Claim是叶子节点排序顺序中排在第一位的有效Claim。也就是说，它在同名的所有Claim要求中有效量最高。

在给定的块中，只有一个Claim可以控制给定的名称。

#### Activation Delay
如果一个stake没有立即成为活动状态，它将在由以下公式确定的块高处成为活动状态:

*ActivationHeight = AcceptedHeight + min(4032, floor( (AcceptedHeight-TakeoverHeight)/32 ))*

此处：

	•	AcceptedHeight是指接受stake时的高度
	•	TakeoverHeight是指名称的控制权变更时的最近高度


在书面形式上，stake生效前的延迟等于接受stake的高度减去最后一次接盘的高度，除以32。这个延迟的上限是4032个区块，也就是7天的区块，每个区块2.5分钟（目标区块时间）。大约需要224天没有接管才能达到最大延迟。

这种延迟的目的是让长期存在的Claim人有时间对变化作出反应，同时保持合理的接管时间，并允许最近或有争议的Claims迅速改变状态。

#### Claim Ordering
为了确定叶子节点中的Claim顺序，采用以下算法：

1. 对每个Claim重新计算有效金额
2. 按有效金额降序对Claim进行排序。对相同金额并列的Claim按块高排序（先低后高），然后按块内的交易顺序排序
3. 如果前一个区块的控制Claim仍然排在第一位，则排序结束
4. 否则，则发生接管。将该名称的接管高度设为当前高度，重新计算现在哪些Claim处于活动状态，并重新进行步骤1和2
5. 此时，有效金额最大的Claim就是这个区块的控制Claim

4的目的是为了处理在不同区块的同一名称上有多个相互竞争的Claim，其中一个Claim成为有效的，但另一个仍未被激活的Claim具有最大的有效金额的情况。步骤4将使较大的Claim也激活，成为控制性Claim。

详见附录中的例子。

#### Normalization
在进行任何比较时，Claim中的名称都被归一化。这是必要的，为避免由于Unicode等价或大小写引起的混淆，这是必要的。当比较名称时，首先使用Unicode规范化表格D（NFD）进行转换，然后使用en_US本地化进行小写。这意味着名称实际上是不区分大小写的。由于竞争同一名称的Claim存储在claimtrie中的同一节点中，因此名称也要进行归一化，以确定claimtrie到该节点的路径。

#### Expiration
在协议的早期版本中，Stake在被接受后会有262974个区块过期（即自动成为放弃）。部署了一个硬分叉，有效地禁用了过期。任何在分叉生效前过期的stake都会被当作被放弃的stake处理。详情请看[请求](https://github.com/lbryio/lbrycrd/pull/137)

### URLs

URLs是对Claims的显性引用，所有的URLs：
1. 包含一个name（参见 Claim属性）
2. 是向一个独立的，具体的Claim的解析

很多Claim和区块链设计的最终目的是提供可记忆的URL，可以在没有区块链完整副本的情况下被客户证明解决（如简化支付验证钱包）。
####	Components
URL是一个带有一个或多个修饰词的名称。单独的裸名会以最新的块高解析到Controlling Claim。下面是一些常见的URL结构。

##### Stream Claim Name
A controlling stream claim.

`lbry://meet-lbry	`
##### Channel Claim Name
频道的claim

`lbry://@lbry` 
##### Channel Claim Name and Stream Claim Name 
一个URL同时包含一个频道和一个流Claim名称。包含两者的 URL 分两步解析。首先，频道被解析为其相关的Claim。然后，解析流Claim名称，从频道中的Claim中获得适当的Claim。

`lbry://@lbry/meet-lbry`
##### Claim ID
该名称与该Claim ID的Claim。允许部分前缀匹配（见URL解析）。
`lbry://meet-lbry:7a0aa95c5023c21c098`
`lbry://meet-lbry:7a`
`lbry://@lbry:3f/meet-lbry`

注意：在本规范的前一个版本中，#字符用于表示url的Claim ID部分。这个字符现在已经被废弃了，以后将不再支持。

##### Sequence
该名称的第n个被接受的Claim。_n必须是一个正数。这可以用来按Claim提出的顺序而不是按支持Claim的信用额度来参考。

`lbry://meet-lbry*1`
`lbry://@lbry*1/meet-lbry`
##### Amount Order 
该名称的第n件Claim，按总额排序(最高的先)。_n必须是一个正数。这对于解决可能成为控制权的非控制权Claim是有用的。

`lbry://meet-lbry$2`
`lbry://meet-lbry$3`
`lbry://@lbry$2/meet-lbry`

##### Query Params
这些参数在LBRY协议中没有意义。它们是供上游应用程序使用的。

`lbry://meet-lbry?arg=value+arg2=value2`

####	Grammar
完整的URL语法是使用[Xquery EBNF](https://www.w3.org/TR/2017/REC-xquery-31-20170321/#EBNFNotation)符号定义的。

```
URL ::= Scheme Path Query?

Scheme ::= 'lbry://'

Path ::=  StreamClaimNameAndModifier | ChannelClaimNameAndModifier ( '/' StreamClaimNameAndModifier )?

StreamClaimNameAndModifier ::= StreamClaimName Modifier?
ChannelClaimNameAndModifier ::= ChannelClaimName Modifier?

StreamClaimName ::= NameChar+
ChannelClaimName ::= '@' NameChar+

Modifier ::= ClaimID | Sequence | AmountOrder
ClaimID ::= ':' Hex+
Sequence ::= '*' PositiveNumber
AmountOrder ::= '$' PositiveNumber

Query ::= '?' QueryParameterList
QueryParameterList ::= QueryParameter ( '&' QueryParameterList )*
QueryParameter ::= QueryParameterName ( '=' QueryParameterValue )?
QueryParameterName ::= NameChar+
QueryParameterValue ::= NameChar+

PositiveDigit ::= [123456789]
Digit ::= '0' | PositiveDigit
PositiveNumber ::= PositiveDigit Digit*

HexAlpha ::= [abcdef]
Hex ::= (Digit | HexAlpha)+

NameChar ::= Char - [=&#:*$@%?/]  /* any character that is not reserved */
Char ::= #x9 | #xA | #xD | [#x20-#xD7FF] | [#xE000-#xFFFD] | [#x10000-#x10FFFF] /* any Unicode character, excluding the surrogate blocks, FFFE, and FFFF. */
```

####	Resolution
	a.	
	b.	
	c.	
	d.	
	e.	
####	Design Notes

### Transactions
	a.	
	b.	
	c.	
	d.	
	a.	
### Consensus
	a.	
	b.	
	c.	
	d.	
	e.	
##	Metadata

### Specification
	
	a.	
### Key Fields
	a.	
	b.	
	c.	
	d.	
	e.	
	f.	
### Channels (Identities)
	a.	
	b.	
	c.	
### Validation



## Metadata
###Specification
###Key Fields
###Channels (Identities)
###Validation
## Data 

###Encoding
###Announce
###Download
###Reflectors and Data Markets

## Appendix 
###Claim Activation Example
###URL Resolution Examples
###Additional Resources

