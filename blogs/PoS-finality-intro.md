# Conflux 的 PoS Finality 与解读

Conflux 近期计划于主网中引入名为 PoS Finality 的机制，本文将就此机制尝试进行解读。

本文为本系列的第一篇文章，希望能讲清楚 PoS Finality 这个概念本身，并讨论 Conflux 为什么要引入这一概念。简而言之，就是讲清楚“是什么”与“为什么”这两个概念。

本文仅代表本人的看法与理解，诸多观点可能存在可讨论之处，如有错误也欢迎各位留言指正。

## 何为 PoS Finality

本文第一部分将简单就 PoS Finality 这个概念进行解释，具体而言，就是分别解释清楚

1. 何为`PoS`
2. 何为`Finality`
3. 何为`PoS Finality`

这三个问题。

### 何为 PoS

> 这一部分大体可看作是 [Conflux杨光：PoW和PoS的全面比较](https://zhuanlan.zhihu.com/p/60702898) 内容的一个子集，原文基本把我能想到的东西都说清楚了，不过因此原文篇幅也偏长。熟悉这一概念的读者可以跳过这一部分。

我们不妨先来回顾一下区块链的核心功能：从一个高层次的角度来看，所有的区块链都可以认为是一个去中心化的账本。“去中心化”意味着这个系统的状态由诸多**节点**一同决定，并且能在一部分节点恶意进行破坏的情况下正常工作；“账本”意味着在所有人眼中，区块链中的交易会有一致的顺序（这个性质也叫**全序性**）。而如何去中心化地维护交易的全序性，就是由区块链中的共识所负责的。

当区块链中的节点身份已知的情况下，可以通过“投票”这一直观的手段来维护账本。这种情况下只要**恶意节点的数目**小于某个阈值（如 1/3），区块链网络就能安全地运行。而问题在于，如果在节点身份未知的互联网环境中维护账本？这时简单地基于节点数进行投票是行不通的，一个恶意的节点可以很容易地实施“女巫攻击”，即控制若干 IP 地址，从而伪造出若干虚假的身份进而突破恶意节点数的阈值。一旦安全阈值被突破，区块链的安全性质将无法被保证。而比特币的一个核心贡献就是使用基于“工作量证明”（`PoW, Proof of Work`）的共识。在比特币网络中，一个节点投入的**算力**越多，它拥有的投票权越大（比特币中，也就是越有可能成功打包区块，获得记账权）。而恶意的节点想要拥有超过安全阈值的算力则是一件困难的事情——这意味着确实地投入算力。

TODO here: graph （describing sibil attack & PoW）

`PoW`所做的事是将难以轻易获取的“算力”映射为了系统中的投票权，从而防止了女巫攻击。而“权益证明”（`PoS, Proof of Stake`）也是顺着这一思路而被提出的。`Stake`指的便是节点在整个系统中拥有的代币数（如以太坊中的 Ether），与`PoW`相似，在`PoS`下，一个节点所拥有的代币数（而非算力）越多，它的投票权越大。`PoS`类共识中，安全阈值变为了**Stake比例**。`PoS`还隐含着这样的逻辑：持有更多代币的角色往往更希望区块链能安全运行，否则一旦区块链的安全性出现问题，币价下跌，持币的角色会遭受到更大的损失。

TODO here graph 传统BFT共识 vs PoW vs PoS

各类基于`PoS`的共识可能会有比较大的差别，这里不作过多讨论。但大体上来说，基于`PoS`的协议回避了`PoW`中“挖矿”时需要消耗大量能源的问题，同时也能达成更高的 TPS 并拥有更低的延迟。但同时， `PoS` 各类协议中，记账的矿工往往在记账前就已经被决定了，这一性质也带来了诸多`PoW`中没有的攻击，从而使得PoS的协议设计更为复杂，不少攻击也难以被完全杜绝。

除此之外，就我个人的角度而言，在公有链系统中，PoS中还存在着一个非常大的问题：如果整个区块链完全由PoS进行驱动，公有链的“自由加入”这一性质便打了折扣。如果一个全新的节点想要参与共识，它必须要想办法从其他矿工手中获取`Stake`，如果其他矿工不愿意出售，那么新的节点将永远无法加入系统。而在PoW类共识中，只要节点能够提供算力（虽然ASIC矿机与通用计算机的效率存在很大差别），它都能随时加入系统。

### 何为 Finality

Finality，常译作“最终性”或“最终确定性”。[币安的词汇表](https://academy.binance.com/zh/glossary/finality)中是这么解释的：

> Finality is the assurance or guarantee that cryptocurrency transactions cannot be altered, reversed, or canceled after they are completed.

可大致意译为：最终性是交易上链后无法被更改、回滚或取消的保证。

在比特币等大部分使用PoW的链（也包括Conflux）中，最终性是概率达成的，换言之，一笔交易上链的时间越长，或者说，该笔交易所位于的区块越“深”，那它被回滚的可能性就越低。当这种可能性低到一定程度，一笔交易就被认为是被“确认”了。比特币中确认时间为6个区块，约一个小时；Conflux中交易在上链一分钟内一般就可以被确认。当一笔交易被确认后，在没有恶意攻击者进行攻击的情况下
在比特币白皮书中基于非常朴素的假设对回滚的概率作出了计算，如下图。

![](image/2021-11-05-15-13-14.png)

其中 q 表示攻击者持有的算力比例，z表示回滚的区块数目，P 表示攻击者区块被回滚的概率。可以看到，攻击者持有的算力比例越高，回滚就越有可能发生。而当攻击者的算力比例突破50%的临界点时，他就能够无视交易所在的区块的深度对其进行回滚。这也是所谓的“51%攻击”。

TODO 攻击者任意进行操作 举例


以下图为例。

比特币定为了6个块（一小时），Conflux的确认概率为1e-8，

TODO： 图 here ，最长链原则+lucky attacker

### 何为 PoS Finality

那么何为 PoS Finality？可以先来参考[论坛](https://forum.conflux.fun/t/conflux-pos-finality/9919)中的解释。

> 在一个 PoW 链生态的早期，在全网算力较低的时候，可能会出现 51% 攻击的问题。特别是公有链的发展催生了一些算力租借平台的时候。在去年，以太经典、Grin 和 Verge 都曾出现了 51% 攻击的问题。

> 为了应对 51% 攻击可能带来的威胁，Conflux 将引入一条独立运行的 PoS 链。PoS 链的共识参与者将定期对树图结构 pivot 区块签名。拥有足够多签名的 pivot 区块应当被所有 PoW 矿工选进 pivot 链，哪怕它的兄弟区块权重更大。简单来说，PoS 链指定了一个 pivot 区块，所有的 PoW 矿工都应该跟随。

> 这意味着，一旦 PoS 共识对一个 pivot 区块投票，即使 51% 攻击者尝试逆转这个区块，也不会被 PoW 节点认可。

> Conflux 要求 PoS 共识克制地使用“指定 pivot 区块”的权力。**一个区块首先要根据 PoW 的规则确认满几分钟，诚实的 PoS 节点才会对它进行签名。这意味着，树图结构的区块排序和确认依然由 PoW 的矿工完成。**

在 Finality 部分已经提到，使用PoW的结构的链中矿工往往会跟随“最长链”（Conflux规则更加复杂），因此如果攻击者能够自行制造一条最长链，就能任意地逆转已上链的交易。

对此Conflux给出的解决办法就是引入一条PoS链，由PoS链不断投票在PoW链上设置checkpoint，从而使得51%攻击者无法大幅度地revert区块。而这一段最核心的内容在我看来是最后一句，表明PoS链的能力与作用：

**一个区块首先要根据 PoW 的规则确认满几分钟，诚实的 PoS 节点才会对它进行签名。这意味着，树图结构的区块排序和确认依然由 PoW 的矿工完成。**

换言之，如果 PoS 链的参与者诚实地执行协议，情况如下：
1. 在PoW链中没有发生51%攻击，那么PoW链的运行情况将与引入 PoS Finality 之前无异：无论是使用ConfluxScan查看交易的状况，还是利用sdk的相关接口操作，都不会有区别。
2. PoW 链中发生了 51% 攻击，产生了回滚
   1. 回滚越过了PoS决定的checkpoint：其他矿工将
   2. 回滚没有越过PoS决定的checkpoint：其他矿工

注：绘图时进行了简化处理，树图结构的PoW链在图中以单链结构表示