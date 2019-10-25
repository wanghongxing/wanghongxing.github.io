---
title: Hyperledger Fabric Key Concepts
date: 2019-04-21 10:56:10
tags:
  - fabric
  - blockchain
---
介绍（Introduction）

Hyperledger Fabric is a platform for distributed ledger solutions underpinned by a modular architecture delivering high degrees of confidentiality, resiliency, flexibility, and scalability. It is designed to support pluggable implementations of different components and accommodate the complexity and intricacies that exist across the economic ecosystem.
Hyperledger Fabric是一个用于分布式账本的解决方案平台，其基础是提供高度机密性、弹性、灵活性和可伸缩性的模块化架构。它的设计目的是支持不同组件的可插入实现，并适应经济生态系统中存在的复杂性和精细之处。

We recommend first-time users begin by going through the rest of the introduction below in order to gain familiarity with how blockchains work and with the specific features and components of Hyperledger Fabric.
我们建议初次用户从下面的介绍开始，以便熟悉区块链是如何工作的，以及Hyperledger Fabric的特定特性和组件。

Once comfortable — or if you’re already familiar with blockchain and Hyperledger Fabric — go to Getting Started and from there explore the demos, technical specifications, APIs, etc.
一旦合适或你已经熟悉了区块链和Hyperledger Fabric，那么你可以从开始着手那里探索演示、技术规范、API等等。

什么是区块链 （What is a Blockchain）?

一种分布式账本（A Distributed Ledger）

At the heart of a blockchain network is a distributed ledger that records all the transactions that take place on the network.
区块链网络的核心是一个分布式帐本，记录网络上发生的所有交易。

A blockchain ledger is often described as decentralized because it is replicated across many network participants, each of whom collaborate in its maintenance. We’ll see that decentralization and collaboration are powerful attributes that mirror the way businesses exchange goods and services in the real world.
区块链帐本通常被描述为分散管理的，因为它在许多网络参与者之间复制，每个参与者在维护工作中协作。我们将看到，分散化和协作是反映真实世界中企业交换商品和服务的强大属性。

在这里插入图片描述

In addition to being decentralized and collaborative, the information recorded to a blockchain is append-only, using cryptographic techniques that guarantee that once a transaction has been added to the ledger it cannot be modified. This property of “immutability” makes it simple to determine the provenance of information because participants can be sure information has not been changed after the fact. It’s why blockchains are sometimes described as systems of proof.
除了分散管理和协作外，记录到区块链的信息是只能添加，使用加密技术可以确保事务一旦添加到帐本中，就不能进行修改。这种“不可变性”的特性使得确定信息的来源变得很简单，因为参与者可以确定信息在事后没有被更改。这就是为什么区块链有时被描述为证明系统。

智能合约 (Smart Contracts)

To support the consistent update of information — and to enable a whole host of ledger functions (transacting, querying, etc) — a blockchain network uses smart contracts to provide controlled access to the ledger.
为了支持信息的一致更新——并支持所有帐本功能(交易、查询等)——区块链网络使用智能合约提供对帐本的受控访问。

在这里插入图片描述

Smart contracts are not only a key mechanism for encapsulating information and keeping it simple across the network, they can also be written to allow participants to execute certain aspects of transactions automatically.
智能合约不仅是封装信息并使其在网络上保持简单的关键机制，还可以编写它们来允许参与者在某些方面自动执行交易。

A smart contract can, for example, be written to stipulate the cost of shipping an item where the shipping charge changes depending on how quickly the item arrives. With the terms agreed to by both parties and written to the ledger, the appropriate funds change hands automatically when the item is received.
例如，一份智能合约可以规定货物的运输成本，运费取决于货物到达的速度。双方同意条款并记入帐本，当货物收到时，相应的资金自动易手。

共识 (Consensus)

The process of keeping the ledger transactions synchronized across the network — to ensure that ledgers update only when transactions are approved by the appropriate participants, and that when ledgers do update, they update with the same transactions in the same order — is called consensus.
使帐本交易在网络上保持同步的过程——确保帐本只有在交易得到适当的参与者批准时才进行更新，以及当帐本进行更新时，它们以相同的排序对相同的交易进行更新——称为共识。

在这里插入图片描述

You’ll learn a lot more about ledgers, smart contracts and consensus later. For now, it’s enough to think of a blockchain as a shared, replicated transaction system which is updated via smart contracts and kept consistently synchronized through a collaborative process called consensus.
稍后你将了解更多关于账本、智能合约和共识的内容。现在，只要将区块链看作是一个共享的、复制的交易系统就足够了，它通过智能合约进行更新，并通过称为共识的协作过程保持一致的同步。

为什么区块链有用 (Why is a Blockchain useful)?

当前的记录系统 (Today’s Systems of Record)

The transactional networks of today are little more than slightly updated versions of networks that have existed since business records have been kept. The members of a business network transact with each other, but they maintain separate records of their transactions. And the things they’re transacting — whether it’s Flemish tapestries in the 16th century or the securities of today — must have their provenance established each time they’re sold to ensure that the business selling an item possesses a chain of title verifying their ownership of it.
当前的交易型网络不过是自保留业务记录以来已经存在的网络的略微更新版本。业务网络的成员彼此进行交易，但他们保持各自的交易记录。他们交易的物品——无论是16世纪的佛兰德挂毯还是今天的证券——每次出售时都必须确定其出处，以确保出售物品的企业拥有一系列产权，以证明其所有权。

What you’re left with is a business network that looks like this:
剩下的就是这样一个商业网络：

在这里插入图片描述

Modern technology has taken this process from stone tablets and paper folders to hard drives and cloud platforms, but the underlying structure is the same. Unified systems for managing the identity of network participants do not exist, establishing provenance is so laborious it takes days to clear securities transactions (the world volume of which is numbered in the many trillions of dollars), contracts must be signed and executed manually, and every database in the system contains unique information and therefore represents a single point of failure.
现代技术已经将这一过程从石板和纸质文件夹转移到硬盘和云平台，但其底层结构是相同的。用统一的系统来管理网络参与者身份并不存在，确立出处是如此费力以致需要很多天去理清证券交易（世界的交易量是数万亿美元)，合同必须手动签署和执行，每个数据库系统中包含独特的信息,因此代表了存在单点故障。

It’s impossible with today’s fractured approach to information and process sharing to build a system of record that spans a business network, even though the needs of visibility and trust are clear.
如今，尽管可见性和信任的需求是明确的，但信息和流程共享的方式是破碎的，要建立一个跨越业务网络的记录系统是不可能的。

区块链差异 (The Blockchain Difference)

What if, instead of the rat’s nest of inefficiencies represented by the “modern” system of transactions, business networks had standard methods for establishing identity on the network, executing transactions, and storing data? What if establishing the provenance of an asset could be determined by looking through a list of transactions that, once written, cannot be changed, and can therefore be trusted?
如果业务网络不再是“现代”交易系统所代表的低效之巢，而是拥有在网络上建立身份、执行交易和存储数据的标准方法？如果可以通过查看一组交易来确定资产的来源，这些交易一旦被写入，就不能更改，因此可以信任呢?

That business network would look more like this:
这个商业网络看起来更像这样:

在这里插入图片描述

This is a blockchain network, wherein every participant has their own replicated copy of the ledger. In addition to ledger information being shared, the processes which update the ledger are also shared. Unlike today’s systems, where a participant’s private programs are used to update their private ledgers, a blockchain system has shared programs to update shared ledgers.
这是一个区块链网络，每个参与者都有自己复制的帐本副本。除了帐本信息共享外，账本的更新过程也是共享的。不像当前的系统，参与者的私有程序被用来更新他们的私有账本，区块链系统已经共享了程序来更新共享账本。

With the ability to coordinate their business network through a shared ledger, blockchain networks can reduce the time, cost, and risk associated with private information and processing while improving trust and visibility.
借助通过共享帐本协调业务网络的能力，区块链网络可以减少与私有信息和处理相关的时间、成本和风险，同时提高信任和可见性。

You now know what blockchain is and why it’s useful. There are a lot of other details that are important, but they all relate to these fundamental ideas of the sharing of information and processes.
现在你知道什么是区块链，以及它为什么有用。还有很多其他重要的细节，但它们都与共享信息和流程的基本思想有关。

什么是 （What is） Hyperledger Fabric?

The Linux Foundation founded the Hyperledger project in 2015 to advance cross-industry blockchain technologies. Rather than declaring a single blockchain standard, it encourages a collaborative approach to developing blockchain technologies via a community process, with intellectual property rights that encourage open development and the adoption of key standards over time.
Linux基金会在2015年建立了Hyperledger项目，以推进跨行业区块链技术。它不是宣布一个单一的区块链标准，而是鼓励通过社区过程协作的方式来开发区块链技术，并拥有鼓励开放开发的知识产权和随着时间推移关键标准的采用。

Hyperledger Fabric is one of the blockchain projects within Hyperledger. Like other blockchain technologies, it has a ledger, uses smart contracts, and is a system by which participants manage their transactions.
Hyperledger Fabric是Hyperledger的区块链项目之一。像其他区块链技术一样，它有一个帐本，使用智能合约，并且是一个参与者管理他们的交易的系统。

Where Hyperledger Fabric breaks from some other blockchain systems is that it is private and permissioned. Rather than an open permissionless system that allows unknown identities to participate in the network (requiring protocols like “proof of work” to validate transactions and secure the network), the members of a Hyperledger Fabric network enroll through a trusted Membership Service Provider (MSP).
Hyperledger Fabric网络与其他区块链系统的不同之处在于它是私有的并且要许可的。Hyperledger Fabric的成员通过受信任的**会员服务提供者(MSP)**注册，而不是允许未知身份参与网络(需要“工作证明”之类的协议来验证交易并保护网络)的开放免许可系统。

Hyperledger Fabric also offers several pluggable options. Ledger data can be stored in multiple formats, consensus mechanisms can be swapped in and out, and different MSPs are supported.
Hyperledger Fabric还提供了几个可插入选项。账本数据可以以多种格式存储，共识机制可以被换进和换去，并且支持不同的MSP。

Hyperledger Fabric also offers the ability to create channels, allowing a group of participants to create a separate ledger of transactions. This is an especially important option for networks where some participants might be competitors and not want every transaction they make — a special price they’re offering to some participants and not others, for example — known to every participant. If two participants form a channel, then those participants — and no others — have copies of the ledger for that channel.
Hyperledger Fabric还提供了创建通道（channels）的功能，允许一组参与者创建单独的交易帐本。对于网络来说，这是一个特别重要的选项，因为有些参与者可能是竞争对手，并不希望他们所做的每一笔交易——例如他们提供给某些参与者的一个特殊价格，而没给其他参与者——让每个参与者都知道。如果两个参与者组成一个通道，那么这两个参与者——而不是其他参与者——就拥有该通道的帐本副本。

共享账本 (Shared Ledger)

Hyperledger Fabric has a ledger subsystem comprising two components: the world state and the transaction log. Each participant has a copy of the ledger to every Hyperledger Fabric network they belong to.
Hyperledger Fabric具有一个帐本子系统，它由两个组件组成：世界状态和交易日志。每个参与者都有一份他们所属Hyperledger Fabric网络的帐本副本。

The world state component describes the state of the ledger at a given point in time. It’s the database of the ledger. The transaction log component records all transactions which have resulted in the current value of the world state; it’s the update history for the world state. The ledger, then, is a combination of the world state database and the transaction log history.
世界状态组件描述了在给定时间点上的帐本状态。这是账本的数据库。交易日志组件记录导致世界状态当前值的所有交易；这是世界状态的更新历史。因此帐本是世界状态数据库和交易日志历史记录的组合。

The ledger has a replaceable data store for the world state. By default, this is a LevelDB key-value store database. The transaction log does not need to be pluggable. It simply records the before and after values of the ledger database being used by the blockchain network.
帐本为世界状态有一个可替换的数据存储。默认情况下，这是一个LevelDB键值存储数据库。交易日志不需要是可插入的。它只记录区块链网络使用的账本数据库的前后值。

智能合约 (Smart Contracts)

Hyperledger Fabric smart contracts are written in chaincode and are invoked by an application external to the blockchain when that application needs to interact with the ledger. In most cases, chaincode interacts only with the database component of the ledger, the world state (querying it, for example), and not the transaction log.
Hyperledger Fabric智能合约是用链代码编写的，当应用程序需要与帐本交互时，由位于区块链外部的应用程序调用。在大多数情况下，链代码只与帐本的数据库组件世界状态(例如查询它)进行交互，而不与交易日志进行交互。

Chaincode can be implemented in several programming languages. Currently, Go and Node are supported.
链代码可以用几种编程语言实现。目前支持Go和Node。（Java在 v1.3版已经支持）

隐私 (Privacy)

Depending on the needs of a network, participants in a Business-to-Business (B2B) network might be extremely sensitive about how much information they share. For other networks, privacy will not be a top concern.
根据网络的需要，B2B网络的参与者可能对他们共享的信息非常敏感。对于其他网络来说，隐私不会是首要问题。

Hyperledger Fabric supports networks where privacy (using channels) is a key operational requirement as well as networks that are comparatively open.
Hyperledger Fabric支持网络，其中隐私（使用通道）是一个关键的操作需求，以及相对开放的网络。

共识 (Consensus)

Transactions must be written to the ledger in the order in which they occur, even though they might be between different sets of participants within the network. For this to happen, the order of transactions must be established and a method for rejecting bad transactions that have been inserted into the ledger in error (or maliciously) must be put into place.
交易必须按照发生的顺序写入帐本，即使它们可能在网络中的不同参与者组之间。要做到这一点，必须建立交易的排序，并且必须采用一种方法来拒绝错误地(或恶意地)插入到帐本中的不良交易。

This is a thoroughly researched area of computer science, and there are many ways to achieve it, each with different trade-offs. For example, PBFT (Practical Byzantine Fault Tolerance) can provide a mechanism for file replicas to communicate with each other to keep each copy consistent, even in the event of corruption. Alternatively, in Bitcoin, ordering happens through a process called mining where competing computers race to solve a cryptographic puzzle which defines the order that all processes subsequently build upon.
这是一个对计算机科学进行了深入研究的领域，有很多方法可以实现它，每个方法都有不同的权衡。例如，PBFT(实用的拜占庭容错)可以提供一种机制，让文件副本相互通信，以保持每个副本的一致性，即使在发生损坏的情况下也是如此。或者，在比特币中，排序是通过一种被称为“挖矿”的过程进行的，在这种过程中，相互竞争的计算机竞相解决一个密码难题，这个难题定义了所有进程随后所建立的顺序。

Hyperledger Fabric has been designed to allow network starters to choose a consensus mechanism that best represents the relationships that exist between participants. As with privacy, there is a spectrum of needs; from networks that are highly structured in their relationships to those that are more peer-to-peer.
Hyperledger Fabric被设计用来允许网络初学者选择一个最能代表参与者之间存在关系的共识机制。与隐私一样，人们的需求也不尽相同;从关系高度结构化的网络到更对等的网络。

We’ll learn more about the Hyperledger Fabric consensus mechanisms, which currently include SOLO and Kafka.
我们将学习更多关于Hyperledger Fabric的共识机制，目前包括SOLO和Kafka。