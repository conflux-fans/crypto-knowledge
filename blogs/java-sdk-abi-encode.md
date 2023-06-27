# Java SDK 合约交互参数如何传递

EVM 智能合约开发过程中，需要频繁的跟合约进行交互，合约交互最主要的工作其实是参数的 ABI 编码及返回结果的 ABI 解码。通常各个语言 SDK 会将编解码操作进行封装，方便合约的交互。弱类型语言 SDK 比如 JavaScript 或 Python 可以直接根据 ABI 文件创建合约交互类。强类型语言则没那么方便，要么使用 CLI 功能直接生成交互类代码，或者根据合约方法描述，手动借助 low level 工具方法，编写编解码函数。

现在我们来看看如何使用 Java 语言来进行 ABI 编解码。

## [Solidity 数据类型](https://docs.soliditylang.org/en/v0.8.10/types.html)

在 Solidity 中值类型如下：

* 布尔值: `bool`
* 整型: `int8-int256`, `uint8-uint256` 以 8 为单位。`int=int256`, `uint=uint256`
* 浮点数：`fixed`, `ufixed` (很少使用)
* 地址类型：`address`
* 固定大小字节数组：`bytes1-bytes32`
* 动态大小字节数组：`bytes`
* 字符串：`string`

引用类型：

* 数组：`T[]`
* 结构体: `struct`
* 映射: `mapping`

## [web3j abi](https://javadoc.io/doc/org.web3j/abi/4.8.8/index.html)

web3j 是以太坊社区开发的一个以太坊 Java SDK, 其中包含一个 [abi package](https://javadoc.io/doc/org.web3j/abi/4.8.8/index.html) 专门用于 ABI 编解码。该 package 提供了用于编解码 Solidity 各种类型的 Java 类，以及编解码类。

![](./image/web3j-abi.png)

## 简单示例

### 参数传递

假设有一个 Solidity 的方法如下:

```js
function newGreeting(string memory _greet) public {
  greet = _greet;
}
```

调用此方法我们需要传递一个 string 类型的参数，在调用部署在链上的合约方法时，需要根据方法的签名生成方法标识符，并且对参数进行 abi encode 得到函数的参数数据，然后将两者连到一块，并作为发送交易的 data 字段发到链上。在 java 里，可以使用 web3j 的 abi package 提供的类和编解码方法类完成此操作：

```java
import org.web3j.abi.DefaultFunctionEncoder;
import org.web3j.abi.datatypes.*;
// 创建 String 对应的 abi.datatypes.Utf8String 类实例
Utf8String str = new Utf8String("hello world"); // string
// 创建 Function 实例
Function f = new Function(
  "newGreeting",
  Arrays.asList(str),
  Collections.emptyList()
);
// 使用默认的编码函数 DefaultFunctionEncoder 对方法进行编码
String abiEncodedData = DefaultFunctionEncoder.encode(f);
// abiEncodedData 即是用于调用合约方法的数据
System.out.println(abiEncodedData);
```

### 返回值 decode

现在来看一下如何解析 Solidity 方法的返回值

```js
function getGreeting() public returns (string memory) {
  return greet;
}
```

可以使用 `DefaultFunctionReturnDecoder` 类的 decode 方法，并指定想要解析成的类型，比如这里是 `Utf8String`, 然后就能得到解析后的数据

```java
static void decodeGreetingResult() {
  String helloWorldData = "0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000b68656c6c6f20776f726c64000000000000000000000000000000000000000000";
  TypeReference stringTypeReference = TypeReference.create(Utf8String.class);
  // 使用  DefaultFunctionReturnDecoder 来解析 data, 
  List<Type> list = DefaultFunctionReturnDecoder.decode(helloWorldData, Arrays.asList(stringTypeReference));
  System.out.println(list.get(0).getValue());
}
```

## abi 编解码详解

### 基础类型参数传递(编码)

```java
import org.web3j.abi.DefaultFunctionEncoder;
import org.web3j.abi.datatypes.*;
import java.util.Arrays;
import java.util.Collections;

Bool b = new Bool(true);  // bool
DynamicBytes db = new DynamicBytes("hello world".getBytes(StandardCharsets.UTF_8));  // bytes
Utf8String str = new Utf8String("hello world"); // string
Uint a = new Uint(BigInteger.valueOf(100)); // uint
Address address = new Address("0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7"); // address
Function f = new Function(
        "methodName",
        Arrays.asList(b, db, str, a, address),
        Collections.emptyList()
);
String abiEncodedData = DefaultFunctionEncoder.encode(f);

// array encode
Address a1 = new Address("0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7");
Address a2 = new Address("0xab3B229eB4BcFF881275E7EA2F0FD24eeaC8C83a");
DynamicArray<Address> da = new DynamicArray<>(Address.class, Arrays.asList(a1, a2));
Function f = new Function(
        "methodName",
        Arrays.asList(da),
        Collections.<TypeReference<?>>emptyList()
);
String abiEncodeAddressData = DefaultFunctionEncoder.encode(f);

// struct encode: assume the struct has two field one is uint, another is address
Uint a = new Uint(BigInteger.valueOf(100));
Address address = new Address("0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7");
DynamicStruct ds = new DynamicStruct(a, address);
Function f = new Function(
        "methodName",
        Arrays.asList(ds),
        Collections.<TypeReference<?>>emptyList()
);
String structAbiEncoded = DefaultFunctionEncoder.encode(f);
```

### 一维数组传参

地址数组传参

```java
// array encode
Address a1 = new Address("0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7");
Address a2 = new Address("0xab3B229eB4BcFF881275E7EA2F0FD24eeaC8C83a");
DynamicArray<Address> da = new DynamicArray<>(Address.class, Arrays.asList(a1, a2));
Function f = new Function(
        "methodName",
        Arrays.asList(da),
        Collections.<TypeReference<?>>emptyList()
);
String abiEncodeAddressData = DefaultFunctionEncoder.encode(f);
System.out.println("address array: " + abiEncodeAddressData);
```

### 结构体传参

假如有一个简单结构体如下：

```js
struct SimpleStruct {
  string name;
  uint256 balance;
  address user;
}
```

可以使用如下方式传参

```java
Utf8String name = new Utf8String("hello world"); // string
Uint balace = new Uint(BigInteger.valueOf(100));
Address user = new Address("0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7");
DynamicStruct ds = new DynamicStruct(name, balance, user);
Function f = new Function(
        "methodName",
        Arrays.asList(ds),
        Collections.<TypeReference<?>>emptyList()
);
String structAbiEncoded = DefaultFunctionEncoder.encode(f);
System.out.println("DynamicStruct: " + structAbiEncoded);
```

也可以先定义一个 Java 类来映射该结构体，然后使用该类来编解码数据:

```java
public static class SimpleStruct extends DynamicStruct {
    public String name;

    public BigInteger balance;

    public String user;

    public SimpleStruct(String name, BigInteger balance, String user) {
        super(new org.web3j.abi.datatypes.Utf8String(name),new org.web3j.abi.datatypes.generated.Uint256(balance),new org.web3j.abi.datatypes.Address(user));
        this.name = name;
        this.balance = balance;
        this.user = user;
    }

    public SimpleStruct(Utf8String name, Uint256 balance, Address user) {
        super(name,balance,user);
        this.name = name.getValue();
        this.balance = balance.getValue();
        this.user = user.getValue();
    }
}
```

```java
String hexAddress = "0x8F03f1a3f10c05E7CCcF75C1Fd10168e06659Be7";
HelloWorld.SimpleStruct ss = new HelloWorld.SimpleStruct("hello", BigInteger.TEN, hexAddress);
Function f = new Function(
        "methodName",
        Arrays.asList(ss),
        Collections.emptyList()
);
String structAbiEncoded = DefaultFunctionEncoder.encode(f);
System.out.println("DynamicStruct: " + structAbiEncoded);
```

### 解析结构体返回值

```java
static void decodeStruct() {
  String structData = "0x00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000a0000000000000000000000008f03f1a3f10c05e7cccf75c1fd10168e06659be7000000000000000000000000000000000000000000000000000000000000000568656c6c6f000000000000000000000000000000000000000000000000000000";
  TypeReference ts = new TypeReference<HelloWorld.SimpleStruct>() {};
  List<Type> list = DefaultFunctionReturnDecoder.decode(structData, Arrays.asList(ts));
  System.out.println(((HelloWorld.SimpleStruct)list.get(0)).user);
}
```

## Conflux SDK 如何传参和解码

`java-conflux-sdk` 对上述方法进行了简单的封装，方便开发者使用。使用 java-conflux-sdk 的 `ContractCall` 调用方法，只需要传递 `methodName` 和 实例好的 ABI 对应类的参数即可

```java
ContractCall cc = new ContractCall(cfx, "cfxtest:aat3bzj1mhgubvfj4psdety7d5x46a9v92gtj86mvj");
Utf8String str = new Utf8String("hello world"); // string
String responseData = cc.call("methodName", str).sendAndGet();
// 使用 DecodeUtil.decode 来解析数据（只能decode 一个返回值）
String result = DecodeUtil.decode(rawData, Utf8String.class);
```

如果合约方法需要改变合约状态，可以使用 `Account.call` 方法，使用方法类似.

```java
Utf8String str = new Utf8String("hello world"); // string
account.call("cfxtest:aat3bzj1mhgubvfj4psdety7d5x46a9v92gtj86mvj", "methodName", str);
```

## Event 编解码

合约的写入操作需要等待交易被打包并执行，因此无法直接获取返回结果，这是可以通过 emit Event 的方式，将数据写入到区块链中，后续可以通过获取`交易 receipt` 或者 `cfx_getLogs|eth_getLogs` 方式获取到这些 log 数据，然后通过 abi decode 解析出来。

关于 Event 的编码方式，可参看 [Solidity 官方介绍](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html?highlight=event#events)。

比如标准的 ERC20 合约会包含 `Transfer` 事件:

```js
event Transfer(address indexed from, address indexed to, uint256 value)
```
该事件的名字为 `Transfer`, 包含两个 `indexed` 参数 `from`, `to`, 以及一个非 indexed 的参数 `value`

该事件产生的 log 是这样的:

```js
{
    "address": "0x3ae298749fac1040d47ce50953dd283b2bd5b3cf",
    "blockHash": "0xfb2aadee632487699f265094d2522306f48347c0fa2b4b1963a3e0544548e6a8",
    "blockNumber": "0x48756",
    "data": "0x0000000000000000000000000000000000000000000000000de0b6b3a7640000",
    "logIndex": "0x0",
    "removed": false,
    "topics": [
        "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0x0000000000000000000000003d69d968e3673e188b2d2d42b6a385686186258f",
        "0x00000000000000000000000068a4275c344548d19e84c680ae2353ea91ed56f9"
    ],
    "transactionHash": "0x6f9a9c6ef287089c03c91077a849fb274cf034a66264a81b131ea2ba6fc106ea",
    "transactionIndex": "0x0",
    "transactionLogIndex": "0x0"
}
```

其中包含：

1. 基本信息：地址，区块hash，交易hash，各种索引
2. topics 数组（最多四条）
3. data 一段数据

其中 topics 最多包含四条内容，`第一条为 Event 的签名`，剩余则为 ABI 编码的 `indexed` 参数，最多三个。而所有非 indexed 参数，则被包含在 data 里面。
因此如果想要解析事件的参数，需要多 topics 和 data 进行 ABI decode 操作。

### indexed 参数解析

可使用 web3j abi 包的 `DefaultFunctionReturnDecoder.decodeIndexedValue` 类来解析 topics 和 data。

比如我们来解析 Transfer 事件 `from`, `to` 参数。此两个参数为 indexed 的，所以位于 topics 中（从第二条开始）.

```java
// 根据事件的参数类型，指定需要解析成 java 类型
TypeReference addressTypeReference = TypeReference.create(Address.class);
String fromTopic = "0x0000000000000000000000003d69d968e3673e188b2d2d42b6a385686186258f"; // topics 的第二条
Address from = (Address) DefaultFunctionReturnDecoder.decodeIndexedValue(fromTopic, addressTypeReference);
System.out.println(from);
// 0x3d69d968e3673e188b2d2d42b6a385686186258f
```

### 非 indexed 参数解析

非 indexed 的参数会被 abi encode 存储于 data 字段中，可使用 `DefaultFunctionReturnDecoder.decode` 来解析

```java
String valueData = "0x0000000000000000000000000000000000000000000000000de0b6b3a7640000";
TypeReference uint256TypeReference = TypeReference.create(Uint256.class);
List<TypeReference<Type>> decodeTypes = Arrays.asList(uint256TypeReference);
List<Type> list = DefaultFunctionReturnDecoder.decode(valueData, decodeTypes);
System.out.println(list.get(0).getValue());
```

## 常见问题

**建议合约的参数和返回结果不要使用复杂数据类型如**：

* 多维数组
* 结构体数组
* 结构体字段类型包含数组，结构体等
