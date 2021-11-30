# 为什么交易发送失败了

在 Conflux 网络，通过 `cfx_sendRawTransaction` 方法发送交易时，如果交易构造不对，发送将会失败。其中一些错误比较常见比如：

* 使用了已被执行过的 nonce
* 使用了已经被发送到交易池中的 nonce

另外还有几种发送失败的情况:

* chainId 使用不匹配
* epochHeight 太大
* gas 超过 1500w (half of block gas limit)
* gas 小余 21000
* data 过大 (超过 200K)
* gasPrice 设置为 0
* 签名错误
* 交易池满

如下是交易发送失败时 `cfx_sendRawTransaction` 方法返回的 RPC 错误

## nonce 使用错误

### 使用了已经被执行的 nonce

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"Transaction 0x4a2cfa73267139d965ab86d41f2af16db09e62ff92a5abffd7f8e743f36f327c is discarded due to a too stale nonce\""
    }
}
```

此种情况需改为当前可以用的（未用的） nonce

### 使用了已经被发送到交易池中的 nonce

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"tx already exist\""
    }
  }
```

或

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "Tx with same nonce already inserted. To replace it, you need to specify a gas price > {}"
    }
}
```

对于这两种情况代表交易已经被发到交易池中了，如果想更新或替换交易的话，可以使用同样的 nonce, 修改对应的字段，并提高 gasPrice 重新发送

### 使用了过大的 nonce

发送交易的 nonce 不能币用户当前 nonce 过大，如果超过 2000 将遇到如下错误:

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"Transaction 0xc875a03e1ce01268931a1a428d8f8313714ab5eb9c2b626bd327af7e5c3e8c03 is discarded due to in too distant future\""
    }
  }
```

## gas

如果交易的 gas 太小(`<21000`)或太大(`>1500w`)会返回如下错误:

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"NotEnoughBaseGas { required: 21000, got: 2000 }\""
    }
}
```

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"transaction gas 20000000 exceeds the maximum value 15000000, the half of pivot block gas limit\""
    }
}
```

## gasPrice

交易的 gasPrice 不能设置为 0:

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"ZeroGasPrice\""
    }
}
```

## data

交易有大小限制，最大不能超过 200k

## epochHeight

如果交易的 epochHeight 跟当前网络的 epochNumber 相比小余超过 10w 会遇到如下错误：

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"EpochHeightOutOfBound { block_height: 53800739, set: 0, transaction_epoch_bound: 100000 }\""
    }
}
```

## chainId 使用错误

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "\"ChainIdMismatch { expected: 1, got: 2 }\""
    }
}
```

## 编码或签名错误

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: raw",
        "data": "\"RlpIncorrectListLen\""
    }
}
```

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "Can not recover pubkey for Ethereum like tx"
    }
}
```

## 交易池满

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "txpool is full"
    }
}
```

或

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "Failed imported to deferred pool: Transaction Pool is full"
    }
}
```

对于此种情况，可等待一会重新发送交易，提高交易的 gasPrice 有助于提高发送的几率

## 其他

### 节点处于 catch-up mode

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32077,
        "message": "Request rejected due to still in the catch up mode.",
        "data": null
    }
}
```

等节点数据同步到最新之后再发送

### 内部错误

```js
{
    "jsonrpc": "2.0",
    "id": "15922956697249514502",
    "error": {
        "code": -32602,
        "message": "Invalid parameters: tx",
        "data": "Failed to read account_cache from storage: {}"
    }
}
```