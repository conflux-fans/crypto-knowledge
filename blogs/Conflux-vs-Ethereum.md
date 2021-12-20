# Conflux 与以太坊的异同

Conflux 是一个兼容 EVM 的高性能公链。通过树图账本结构和 GHAST 共识算法将 TPS 提高到 3000。账本和共识的不同也导致 Conflux 与以太坊有一些区别。

## 账本结构区别

Conflux 账本采用[树图结构](https://confluxnetwork.org/files/Conflux_Technical_Presentation_20200309.pdf)，以太坊的账本则是链式结构。

## [地址格式区别](./conflux-address.md)

Conflux 采用 base32 编码地址：

```cfx:aarc9abycue0hhzgyrr53m6cxedgccrmmyybjgh4xg```

以太坊地址为 hex40 格式：

```0x1016f75c54c607f082ae6b0881fac0abeda21781```

## RPC 方法区别

Conflux 实现了以  [`cfx` 开头的 RPC 方法](http://developer.confluxnetwork.org/conflux-doc/docs/json_rpc/)，而[以太坊的 RPC 均以 `eth` 开头](https://eth.wiki/json-rpc/API)

## SDK 区别

因为 RPC 方法的不同，导致以太坊各种语言的 SDK(ethers.js, web3.js, web3j)，无法在 Conflux 网络使用，因此 Conflux 网络单独提供了 SDK：

* [js-conflux-sdk](https://docs.confluxnetwork.org/js-conflux-sdk)
* [go-conflux-sdk](https://github.com/conflux-chain/go-conflux-sdk)
* [java-conflux-sdk](https://github.com/conflux-chain/java-conflux-sdk)

## 钱包工具的区别

Conflux 网络提供了专门的钱包和开发工具.

* [Conflux Portal](https://portal.confluxnetwork.org/) 是一款同 MetaMask 类似的 Conflux 网络浏览器插件
* [Conflux-Truffle](https://www.npmjs.com/package/conflux-truffle) 是在 Truffle 的基础上对 Conflux 网络进行了适配而实现的一款 Conflux 网络智能合约开发工具
* [ChainIDE Conflux](https://chainide.com/s/createTempProject/conflux?language=en) 是一款在线 Solidity 智能合约开发环境，对应于以太坊的 Remix

## EVM 区别

Conflux 智能合约 VM 实现了与 EVM 的兼容，机会所有以太坊智能合约可以直接编译并部署到 Conflux 网络。但因为账本结构的不同，两者还是有一点点区别，具体参看[此介绍](https://juejin.cn/post/6879964152627101709)。

另外 Conflux 智能合约地址生成规则与以太坊也不同，地址生成受三个因素影响：

* 部署交易 from 地址
* 部署交易 nonce
* 合约代码

而以太坊只受 from + nonce 影响
