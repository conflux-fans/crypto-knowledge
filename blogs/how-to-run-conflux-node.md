# 如何运行一个 Conflux 节点

Conflux 是一个基于 PoW(工作量证明) 的完全去中心化网络，如果想要参与此去中心化网络挖矿，或者拥有自己的 RPC 服务需要自己运行一个 node (也称 client)。本文将介绍如何运行一个 Conflux 节点。

## Archivenode VS fullnode

Conflux 的节点分为三种类型：归档节点(archivenode)，全节点(fullnode)，轻节点(lightnode)。不同类型节点的区别在于保留存储的数据量不同，归档节点最全，轻节点最少。当然存储数据越多消耗的系统硬件资源越多。关于不同类型节点的详细介绍[参看这里](https://juejin.cn/post/6854573216930693134)

通常情况下如果想参与挖矿,运行一个全节点即可，如果想作为 RPC 服务来使用则需要运行一个 Archivenode. 轻节点则主要用于作为钱包来使用。

## 机器配置

运行一个 archivenode 的机器资源大致如下：

* CPU：`4Core`
* 内存：`16G`
* 硬盘：`500G`

fullnode 对机器配置的要求会低一些，如果想参与挖矿出块的话，需要有单独的`显卡`。

另外: 建议将系统的最大文件打开数调高到 `10000`。一般 Linux 系统默认为 1024, 不太够用。

阿里云建议使用通用型g7或性能更强配置；AWS推荐m5.xlarge或性能更强配置。

作为rpc节点时，最好使用高性能磁盘：阿里云推荐磁盘使用ESSD盘；AWS至少使用gp3类型，不低于6000iops，或者使用Provisioned IOPS SSD按需求自定义。

## 如何获取节点程序和配置

Conflux 网络节点程序的获取方式，首推到官方 Github [Conflux-rust](https://github.com/conflux-chain/conflux-rust) 仓库的 [Release](https://github.com/Conflux-Chain/conflux-rust/releases) 页面进行下载, 一般直接下载最新 Release 的版本即可。每个 Release 的版本不仅包含源代码，还提供 Windows, Mac, Linux 三大平台预编译好的节点程序。

![](image/conflux-release-page.png)

**需要注意**的是目前主网和测试网节点程序的版本发布是两条线: 主网一般是 `Conflux-vx.x.x`, 测试网则为 `Conflux-vx.x.x-testnet`. 下载程序时需要根据个人的需求选择正确的版本线。

下载的 zip 包，解压后是一个 run 文件夹，里面包含如下内容：

```sh
➜  run tree
.
├── conflux  # 节点程序
├── log.yaml # 日志配置文件
├── start.bat # windows 启动脚本
├── start.sh # unix 启动脚本
├── tethys.toml # 主网配置文件 v2.0 的配置文件改名为 hydra.toml
└── throttling.toml # 限流配置文件

0 directories, 6 files
```

其中主要文件为 `conflux` 和 `tethys.toml (或 hydra.toml)`, 如果下载的是 windows 包的话, 节点程序为 `conflux.exe`

另外一种方式是从源码编译节点程序，如果有兴趣的话，可以[参考此文档](https://developer.confluxnetwork.org/conflux-doc/docs/installation)自行编译。

## 主要配置项

在运行节点前需要先准备好节点配置文件，在下载的程序包里可以找到配置文件，一般主网是 `tethys.toml (或 hydra.toml)`, 测试网则为 `testnet.toml`. 两个配置文件主要区别在于 `bootnodes` 和 `chainId` 的配置值不同。开发者也可以到 `conflux-rust` Github 仓库的 `run 目录`下面查找配置文件。文件名同样为 [`tethys.toml`](https://github.com/Conflux-Chain/conflux-rust/blob/master/run/tethys.toml) 或 `testnet.toml`。

**通常情况用户不需要修改任何配置，直接运行启动脚本即可**(不想了解配置细节？直接跳到下一章节运行节点)。但如果想打开某些功能或自定义节点某些行为，就需要自行设置一些配置参数，以下为最常用的一些配置：

### 节点类型

* `node_type`: 用于设置启动节点的类型，可选值为 `full` (默认值), `archive`, `light`

### chainId

* chainId 用于配置节点要连接的链的ID，主网为 1029, 测试网为 1 (一般不需要修改)

### Miner related

* `mining_address`: 节点挖矿奖励接收地址，可以配置 hex40 地址或 CIP-37 地址(注意：地址的 network prefix 需要与当前配置的 chainId 一致)，如果配置了该项 `minint_type` 默认为 `stratum`
* `mining_type`: 可选值为 `stratum`, `cpu`, `disable`
* `stratum_listen_address`: stratum 地址
* `stratum_port`: stratum 端口号
* `stratum_secret`: stratum 连接凭证

### RPC related

* `jsonrpc_cors`: 用于控制 rpc 域名验证策略，可选值为 `none`, `all`, 或者逗号(无空格)分隔的域名
* `jsonrpc_http_keep_alive`: `false` or `true` 用于控制是否为 rpc HTTP connections 设置 KeepAlive 
* `jsonrpc_ws_port`: websocket rpc 端口号
* `jsonrpc_http_port`: http rpc 端口号
* `public_rpc_apis`: 对外开放访问的 rpc api，可选值为 `all`, `safe`, `txpool`, `pos`, `cfx`, `debug`, `pubsub`, `test`, `trace` (safe=cfx+pubsub)。一般建议设置为 `safe`
* `persist_tx_index`: `true` or `false` 如果需要处理 transaction 相关 RPCs 的话，需要同时打开此配置，不然将只能访问到最近的交易信息
* `persist_block_number_index`: `true` or `false` 如果想要通过 blockNumber 查询 block 信息，需要打开此配置
* `executive_trace`: `true` or `false` 是否打开 trace EVM execution 功能，如果打开 trace 会被记录到 database 中
* `get_logs_filter_max_epoch_range`: Event log 获取方法 `cfx_getLogs` 调用，对节点性能影响很大，可以通过此选项配置 该方法一次能查询的 epoch 范围最大值
* `get_logs_filter_max_limit`: `cfx_getLogs` 方法一次查询能够返回 log 数量的最大值

### Snapshot 

* `additional_maintained_snapshot_count`: 用于设置 stable checkpoint 之前 snapshot 需要保留的个数，默认为 0， stable genesis 之前的 snapshot 都会被删掉。如果用户想查询比较久远的历史状态，需要设置此选项。此选项开始后，磁盘用量同样会增加许多。

### directries

* `conflux_data_dir`: 数据（block data, state data, node database）的存放目录
* `block_db_dir`: block 数据存放目录，默认情况会存放到 conflux_data_dir 指定目录下的 blockchain_db 目录中
* `netconf_dir`: 用于控制 network 相关的持久化目录，包括 `net_key`

### Log related

* `log_conf`: 用于指定 log 详细配置文件如 `log.yaml`，配置文件中的设置会覆盖 `log_level` 设置
* `log_file`: 指定 log 的路径，不设置的话会输出到 stdout
* `log_level`: 日志打印的级别，可选值为 `error`, `warn`, `info`, `debug`, `trace`, `off` 

日志的 log 级别越高，打印的日志越多，响应的会占用贡多的存储空间，也会影响节点的性能.

### 开发者(dev)模式

Conflux-rust 还提供一个开发者(dev)模式，该模式下会启动一个单节点链，并默认打开所有的 RPC 方法。此模式节点非常适合智能合约开发者快速部署和调试合约。
其配置方式如下：

* `bootnodes`: 注释掉此配置
* `mode`: 将模式选项配置为 `dev`
* `dev_block_interval_ms`: 出块间隔时间, 单位为毫秒(ms)

### 配置 genesis 账户

在 dev 模式下可以通过一个单独的 `genesis_secrets.txt` 文件，配置 genesis 账户，该文件中需要一行放置一个私钥（不带0x前缀）, 并在配置文件中添加 `genesis_secrets` 配置项，将值配置为 该文件的路径:

```toml
genesis_secrets = './genesis_secrets.txt'
```

这样节点启动之后，每个账户初始会有 `10000,000,000,000,000,000,000` Drip 也就是 1w CFX。

### 其他

* `net_key`: 是一个 256 bit 的私钥，用于生成唯一 node id，该选项如果不调会随机生成，如果设置可以填一个长度为 64 的 hex 字符串
* `tx_pool_size`: 交易允许存放的最大交易数(`默认 50W`)
* `tx_pool_min_tx_gas_price`: 交易池对交易 gasPrice 的最小限制(`默认为 1`)

关于完整的配置项，可以直接查看配置文件，其中有所有的可配置项，以及详细的注释介绍.

## 运行节点

配置文件配好了，就可以通过节点程序，运行节点了。
```sh
# 运行启动脚本
$ ./start.sh
```

如果你在 stdout 或日志文件看到如下内容，表示节点已经成功启动了:

```
2021-04-14T11:54:23.518634+08:00 INFO  main                 network::thr - throttling.initialize: min = 10M, max = 64M, cap = 256M
2021-04-14T11:54:23.519229+08:00 INFO  main                 conflux      -
:'######:::'#######::'##::: ##:'########:'##:::::::'##::::'##:'##::::'##:
'##... ##:'##.... ##: ###:: ##: ##.....:: ##::::::: ##:::: ##:. ##::'##::
 ##:::..:: ##:::: ##: ####: ##: ##::::::: ##::::::: ##:::: ##::. ##'##:::
 ##::::::: ##:::: ##: ## ## ##: ######::: ##::::::: ##:::: ##:::. ###::::
 ##::::::: ##:::: ##: ##. ####: ##...:::: ##::::::: ##:::: ##::: ## ##:::
 ##::: ##: ##:::: ##: ##:. ###: ##::::::: ##::::::: ##:::: ##:: ##:. ##::
. ######::. #######:: ##::. ##: ##::::::: ########:. #######:: ##:::. ##:
:......::::.......:::..::::..::..::::::::........:::.......:::..:::::..::
Current Version: 1.1.3-testnet

2021-04-14T11:54:23.519271+08:00 INFO  main                 conflux      - Starting full client...
```

节点启动后会在 run 目录里新建两个文件夹 `blockchain_data`, `log` 用于存储节点数据和日志。

启动一个全新的主网或测试网节点后，它会从网络同步历史区块数据，追赶中的节点处于 catch up 模式，可以从日志看到节点的状态和最新的 epoch 数：
```
2021-04-16T14:49:11.896942+08:00 INFO  IO Worker #1         cfxcore::syn - Catch-up mode: true, latest epoch: 102120 missing_bodies: 0
2021-04-16T14:49:12.909607+08:00 INFO  IO Worker #3         cfxcore::syn - Catch-up mode: true, latest epoch: 102120 missing_bodies: 0
2021-04-16T14:49:13.922918+08:00 INFO  IO Worker #1         cfxcore::syn - Catch-up mode: true, latest epoch: 102120 missing_bodies: 0
2021-04-16T14:49:14.828910+08:00 INFO  IO Worker #1         cfxcore::syn - Catch-up mode: true, latest epoch: 102180 missing_bodies: 0
```

你也可以通过 `cfx_getStatus` 方法获取当前节点的最新 epochNumber，并跟 scan 的最新 epoch 比较从而判断数据是否已经同步到了最新。

### RPC 服务

节点启动之后，并且打开了 RPC 相关的端口号和配置的话，则钱包，Dapp 可以通过 RPC url 访问节点. 例如

```http://node-ip:12537``` 

Portal 钱包中添加网络，或者 SDK 实例的时候可以使用此地址.

## 使用 Docker 运行节点

对 Docker 比较熟悉的小伙伴也可以使用 Docker 来运行一个节点。官方提供了各个版本的 [Docker image](https://github.com/conflux-chain/conflux-docker) 可以自行 pull image 并运行。

因为节点数据比较大，所以建议在运行 image 时，挂载一个数据目录用于存放节点数据。

目前发布的镜像 tag 有三条 pipeline:

* `x.x.x-mainnet`: 主网镜像
* `x.x.x-testnet`: 测试网镜像
* `x.x.x`: 开发模式镜像，此模式下会自动初始化十个账号，可用于本地快速开发

## Conflux v2.0 Updates

Conflux v2.0 (Hydra) 是一个重大版本升级，通过 8 个不同的 CIP 引入了一系列新功能。其中最重要的是 PoS 机制和 eSpace 空间。该版本升级也给节点的运营带来一些变化

### 配置

配置文件命名从 `tethys.toml` 改为了 `hydra.toml`

#### 新增配置项

* `evm_chain_id = 1030` eSpace 的 chainId，主网为 `1030`，测试网为 `71`
* `jsonrpc_http_eth_port = 8545` eSpace http 端口号
* `jsonrpc_ws_eth_port = 8546` eSpace websocket 端口号
* `public_evm_rpc_apis = "evm"` eSpace RPC 打开的 namespace
* `dev_pos_private_key_encryption_password = "your-password"` (可选) 引入 PoS 机制后，节点启动时需要配置一个用于保护 pos_key 的密码，该密码可通过命令行交互方式设置，也可以在配置文件中通过 `dev_pos_private_key_encryption_password` 配置项配置。注意：该配置项在配置文件中是明文配置的，可能会有泄漏风险，请妥善保护。

另外配置文件中的 `bootnodes` 配置也进行了更新。

### pos_config 目录

`V2.0 hardfork 后`，下载的程序包中会包含一个新的目录 pos_config 该目录中存放 PoS 链的 genesis 信息包含三个文件

* `genesis_file`
* `initial_nodes.json`
* `pos_config.yaml`

节点运营者无需对这些文件做任何处理，只要确保运行节点的目录中包含 `pos_config` 文件夹即可.

**节点首次启动会在该目录下创建一个 `pos_key`, 该文件存储了节点参与 PoS 的私钥，请妥善保管，不要错误删掉或意外泄漏**

注意：若是参与V2.0 的升级`过程`则会经历两步，第一步直接升级 binary 程序和更新配置文件，这时无需添加 pos_config 文件夹。第二步则是添加 pos_config 文件夹

### 节点启动

节点的启动方式没有发生变化，还是通过 `conflux` binary 程序加配置文件启动。若配置文件中未配置 `dev_pos_private_key_encryption_password` 配置项，则节点第一次启动时，会要求设置 `pos_key` 密码, 以后节点每次重启也需要通过命令行输入密码。若配置文件中加了此配置项，则启动体验跟之前保持一致。

### 节点硬盘要求

由于 Conflux 网络已运行超过 1 年时间，节点数据增多不少，因此节点机器的建议硬盘大小从 200G 调整为 500G

### PoS 节点运行注意事项

如果参与到 PoS 机制当中（获取 PoS 收益），需要运行一个 PoS 节点，运行 PoS 节点需要注意一下事项

* 节点的 `pos_key` 文件中包含节点参与 PoS 的私钥，需要妥善保管，不能丢失和泄漏
* `pos_key` 文件千万不能多节点复用，此操作会导致 CFX 被锁进 PoS 中永远无法取出
* 若节点被强制退出 (foreRetired) 则该节点在 PoS 中的所有票都会被 unlock，unlock 期最长可达 14天，此期间没有 PoS 收益

### PoS 节点更新或重启如何避免被 forceRetired

PoS 投票节点重启防止被强制退出的操作流程：

1. 在PoS节点上运行`./conflux rpc local pos stop_election`，会返回null或者返回一个未来的PoS区块编号。此时节点不再申请加入PoS委员会。
2. 如果返回了区块编号，则保持节点运行。在返回的PoS区块已经生成之后（几个小时后），再次运行相同命令，此时应该返回null。在这个区块之后节点不再获得PoS奖励。
3. 如果命令返回值为null，则节点可以安全关闭。在节点重启之后会自动再次加入PoS投票过程（需要2-3个小时才会获得新的PoS挖矿奖励）。

## 常见问题

### 为什么重启后，同步需要很久？

节点重启后会从上个 checkpoint 开始同步，并重新 replay 区块数据，根据当前距离上一 checkpoint 的远近，需要等待不同的时长，才能开始从最新区块开始同步.
这是正常现象，一般会等几分钟到十几分钟不等。

### 为什么节点同步的区块高度卡住，不再增长?

如果发现区块同步卡住，不再增长。可查看日志或终端是否有错误，如果没有错误大概率是因为网络原因，可尝试重启节点。

### 修改配置后，重启节点需要清除数据么?

分情况，有的需要，有的不需要。如果修改的配置涉及到数据存储或数据索引，需要清数据重启节点，比如:

* `persist_tx_index`
* `executive_trace`
* `persist_block_number_index`

修改其他配置不需要清数据，直接重启即可.

### 目前的 archive node 数据有多大?

截止到 2021.11.04 区块数据的压缩包大小为不到 90 G

### 如何参与挖矿?

挖矿需要使用 GPU 参与，具体可参看[这里](https://forum.conflux.fun/t/conflux-tethys-gpu-mining-instruction-v1-1-4/3775)

### 如何快速同步数据，从而运行一个 archive node 

可使用 [fullnode-node](https://github.com/conflux-fans/fullnode-tool) 下载归档节点的数据快照，使用快照的节点数据，可以快速同步到最新数据。

### 节点运行 error 日志怎么看?

如果是通过 `start.sh` 运行的节点，可以在相同目录中的 `stderr.txt` 查看错误原因。

### 运行节点是否需要公网 IP？

如果不对外提供 RPC 服务，运行一个 Conflux 节点不需要公网IP。

### 节点升级 v2 错误

#### No such device or address

若节点从 v1.x 版本升级到 v2 遇到如下错误，则可能节点是通过 Supervisor 等方式启动. 节点启动时无法正确读取 pos_key 加密密码。
此种情况可在配置文件中添加配置项 `dev_pos_private_key_encryption_password`

```sh
Error: "failed to start full client: \"Os { code: 6, kind: Uncategorized, message: \\\"No such device or address\\\" }\""
```

## 参考

* [官方运行节点文档](https://developer.confluxnetwork.org/run-a-node/en/how_to_get)
* [节点程序源码](https://github.com/conflux-chain/conflux-rust)
