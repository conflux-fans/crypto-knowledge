# 如何升级 PoS 节点

Conflux-rust v2.0 引入了 PoS 机制，用于提高网络的 finality，从而提高网络的安全性，并且 CFX 持有者可以通过参与 PoS 获取一定（至少年化10%以上）的收益。
参与 PoS 机制需要运行一个 fullnode/archivenode, 并将 Conflux 账户地址与该节点进行绑定。绑定之后 CFX 持有者可以通过与 PoSRegister 内置合约交互增加或减少投票。
获取的收益将直接发送至绑定的 Conflux PoW 账户地址。

PoS 节点运营者，需要保证 PoS 节点的稳定性，节点越稳定收益越高。

## 运行 PoS 节点的风险

### forceRetired

如果 PoS 节点`当选`成为了委员会委员，但又`没有正常履行`委员的职责（参与 PoS 投票），且持续超过`一定时间`（一小时），则节点将会被强制退休(forceRetired)。
通常以下三种情况会导致节点被强制退休：

- 节点宕机
- 区块数据同步落后
- 节点重启时间过长

如果节点被强制退休，该节点的所有 PoS 投票会被自动进行解锁操作，解锁的票需要经过 `7-14` 天的时间才会变成`解锁状态`，解锁的票无法获取 PoS 收益。
节点的强制退休状态会`持续 7 天`，七天之后才会变为正常状态。

节点发生强制退休，其质押的 CFX 本金不会丢失，只是会被锁定 7-14 天，期间无法获取收益。

### CFX 被罚没

参与 PoS 共识，只有一种情况 CFX 会被罚没：同一个节点参与投票时，对同一个投票选项（比如PoS 区块）投出了不同的票，此种情况节点会被认为恶意攻击 PoS 共识。
该节点所质押的所有 CFX 将会被`永久锁死，无法取出`.

导致此种情况发生只有一种 case：两个 PoS node 使用同一个  pos_key 同时参与 PoS 共识。

## 运营 PoS 节点注意事项

运行 PoS 节点除了正确的配置 pos_config 并运行节点外，需要注意一下事项：

- 保证节点机器有足够的硬盘空间，建议提供 500GB 以上空间
- 妥善保管节点的 pos_key 以及 pos_key 的密码
- 增大系统的`最大文件打开数` 到 65536

除此之外建议增加以下监控：

- 节点同步区块是否落后：可是用本地节点最新 epochNumber  与官方节点的 epochNumber 比较
- 节点是否正常参与 PoS 投票：可通过 PoS RPC 获取 PoS 的投票交易来判断

另外如果想重新生成 `pos_config/pos_key` 文件，需要改将 `pos_db/secure_storage.json` 文件一并删除，然后重启即可。

## 如何升级节点保证不被强制退休

PoS 节点非不必要，不建议进行升级操作，如果的确需要进行升级操作，建议在升级前，先通知节点停止参与选举，待节点状态变为不参与状态后再关停节点并进行升级操作。
节点重启后会自动重新开始参与 PoS 选举。具体操作步骤如下：

1. 运行命令 `./conflux RPC local PoS stop_election` 通知节点停止参与选举，但节点不会立刻结束参与选举，整个停止参选过程可能需要几个小时。该命令会返回节点停止`参与投票的区块号`
2. 每隔一段时间重复执行此命令以查看节点的状态，如果命令返回为 null 则节点已完成停止参与投票，此时可进行节点关闭操作。
3. 升级完成后重启节点，节点会自动重新参与投票。（节点重启时会自动重新执行从最近 checkpoint 到最新的区块，此过程可能持续较长（几个小时），也可能很快）

## 如何实现 PoS 节点不停机更新

Conflux-rust 最新版本 `v2.0.1` 允许节点启动时不参与 PoS 投票，并通过命令控制节点停止/开始参与投票，通过此特性可以实现节点的宕机切换和无缝升级，从而保证 PoS 节点的持续运行。

**注意：此模式下主备两台机器会使用同一个 pos_key, 因此操作时千万不能让两台机子同时参与 PoS 投票，否则会有 CFX 被锁死的风险。**

该模式是通过 [PR2438](https://github.com/Conflux-Chain/conflux-rust/pull/2438) 添加进来的，在 PR 中有主备升级操作的英文介绍。大致步骤如下:

假设已经获取了 `pos_config/pos_key` 文件

- 准备两个机器 A， B 均配置 `pos_started_as_voter=false` 选项，并使用相同的 `pos_config/pos_key`，然后让两个节点运行起来，此时这两个节点均不会参与 PoS 共识
- 可以通过命令 `./conflux rpc local pos voting_status` 获取节点的投票状态，如果返回 true 表示节点在参与投票， false 表示未参与投票
- 通过第二步的命令确认两个节点均未参与 PoS 的状态后，可以通过命令 `./conflux rpc local pos start_voting` 让其中一个节点开始参与 PoS 共识

此时我们搭建了两个节点，一主一备，并且只有一个节点参与 PoS 共识，如果需要进行节点升级可进行主备切换操作：

- 运行命令 `./conflux rpc local pos stop_voting` 停止主节点参与 PoS 共识，可通过状态获取命令检查是否停止成功。投票停止后该节点会把文件 `pos_db/secure_storage.json` 重命名为 `pos_db/secure_storage.json.save`
- 将此文件移动到备节点的 `pos_db` 目录，此时该目录同时包含 `pos_db/secure_storage.json` 和 `pos_db/secure_storage.json.save` 两个文件
- 运行命令 `./conflux rpc local pos start_voting` 让备节点开始参与 PoS 共识，此时节点会自动使用  `pos_db/secure_storage.json.save` 替换 `pos_db/secure_storage.json` 文件。(注意执行此步时，一定要确保主节点处于停止参与 PoS 共识的状态)

至此我们就完成了主备切换操作，此时可对为参与共识的节点进行升级操作。

**再次提醒: 进行主备切换一定要再三检查确保主节点已停止参与 PoS 共识**

如果主节点直接宕掉了，可以手动把节点的 `pos_db/secure_storage.json` 重命名为 `pos_db/secure_storage.json.save`，并复制到备节点的 pos_db 目录中，然后让备节点开始参与投票。