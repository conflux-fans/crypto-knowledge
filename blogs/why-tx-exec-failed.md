# 交易执行失败的常见原因

成功发送并被打包的交易并不一定会执行成功，交易执行有成功，失败，Skip(跳过) 三种情况，通过交易的 `status` 字段或交易 Receipt 的 `outcomeStatus` 字段可以判断交易的最终执行结果状态

* 0: 成功
* 1: 失败
* null: 交易被跳过 (交易所在的区块已被执行，但交易的 status 仍未 null 的情况)

**注意：跟以太坊相反**

执行失败的交易，其 `receipt` 中的 `txExecErrorMsg` 字段会包含一些执行失败的原因或错误信息

## 可能的错误消息类型

### `VmError(OutOfGas)`

交易所指定的 `gas` 小余交易执行实际所用的 gas 数量，因而失败。

### `VmError(ExceedStorageLimit)`

交易所指定的 `storageLimit` 小余交易执行实际所占用的存储空间，因而失败。

### `NotEnoughCash`

交易发送账户余额不够，导致交易执行失败

### `Vm reverted, xxxx`

合约执行失败，一般是没有通过合约的 `require` 检查语句，require 语句的错误 message 会被包含在此种错误信息里

### `VmError(BadInstruction`

合约部署交易的 `data` 设置错误，可能是构造函数参数没有正确设置导致

### `Vm reverted, `

合约执行失败，但合约没有提供详细的失败信息

## 错误消息 changlog

[Error message changed since v1.1.4](https://github.com/Conflux-Chain/CIPs/issues/70)

## Trace returnData

除了 receipt 的 `txExecErrorMsg` 字段，还可以通过 transaction 的 trace 了解交易执行失败的原因。
如果某个 trace 的类型是 `call`/`create` `result` 并且其 `outcome` 是 `fail` 则 trace 的 `returnData` 即是合约执行失败的错误信息（hex 编码的）