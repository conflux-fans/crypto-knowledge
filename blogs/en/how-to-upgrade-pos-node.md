# How to Upgrade PoS Nodes

Conflux-rust v2.0 introduced a PoS mechanism to increase the finality of the network, in order to improve the security of the network and allow CFX holders to receive a certain amount of interest (at least 10% annualized) by participating in PoS.
To participate in the PoS consensus, you need to run a full node / archive node, and bind the Conflux account to the node. After binding, CFX holders can increase or decrease their votes by interacting with the PoSRegister internal contract.
Earnings will be sent directly to the bound Conflux PoW account address.

PoS node operators should maintain the stability of the PoS nodes. The more stable the nodes are, the higher the revenue is.

## Risks of running PoS nodes

### forceRetired

If a PoS node is `elected` to the committee, but then does not `properly perform` the duties of a member (i.e. participate in PoS voting) for more than `a certain amount of time` (one hour), the node will be forceRetired.

There are normally three conditions leading to a node being force retired:

- The node is down
- The node is out of sync
- The node is taking too much time to reboot

If a node is forceRetired, all PoS vote rights of this node will be automatically unlocked. It will take `7-14` days for them to become `unlocked`. The unlocked votes will not get PoS revenue.
A node's forced retirement status will last for 7 days. After that, it will change to normal status.

When a node is forceRetired, the staked CFX principal will not be lost, but locked for 7-14 days, during which time it will not be able to receive interest.

### CFX confiscated

There is only one scenario where CFX will be confiscated: if a node participates in the PoS consensus and votes differently on the same option (e.g. PoS block). In this case, the node is considered as a malicious attacker to the PoS consensus.
All CFX staked by the node will be `permanently locked up and cannot be retrieved`.

The only case for this scenario to take place is that two PoS nodes are using the same pos_key to participate in PoS consensus at the same time.

## Notes on running PoS nodes

In addition to configuring pos_config correctly, the following considerations should be taken into account when running a PoS node.

- Make sure the machine running the node has enough hard disk space. Recommended amount: at least 500GB
- Keep the pos_key and its password safe.
- Increase the operating system variable `file-max` to 65536, in order to prevent the "too many open files" error.

In addition, it is recommended to additionally monitor the following conditions:

- Whether the node is behind the block synchronization: compare the latest epochNumber of the local node with the epochNumber of the official node.
- Whether the node is participating in PoS voting properly: this can be determined by getting the PoS voting transactions through PoS RPC.

If you want to regenerate the `pos_config/pos_key` file, you need to delete both the key file and `pos_db/secure_storage.json` and restart the system.

## How to upgrade nodes without being forcefully retired

It is not recommended to upgrade PoS nodes if not necessary. If you do need to upgrade the node, it is recommended to notify the node to stop participating in the election before upgrading. After the node status changes to non-participating, you can shut down the node and upgrade it safely.
The node will automatically rejoin the PoS election after restarting. Specific steps are as follows:

1. Run the command `./conflux RPC local PoS stop_election` to inform the node to stop participating in the election. The node will not end its participation immediately, and the whole process may take several hours. The command returns the `block number` where the node will stop participating in the election.
2. Repeat this command to check the status of the node. If the command returns `null`, then the node has stopped participation in the election. It can be shut down at this time.
3. Restart the node after the upgrade is completed. The node will automatically re-participate in voting. (Restarting a node will automatically re-execute from the latest checkpoint to the newest block. It may take a long time (several hours), but could be very fast as well)

## How to upgrade PoS nodes without downtime

The latest version of Conflux-rust `v2.0.1` allows nodes not to participate in PoS voting at startup, and control nodes to stop/start participating in voting by commands. This feature enables nodes to perform downtime switchover and seamless upgrades, in order to ensure the continuity of PoS nodes.

**Note: Both machines in this mode will use the same pos_key, so you must not let both machines participate in PoS polling at the same time, as there will be a risk of CFX lockup.**

This feature is added by [PR2438](https://github.com/Conflux-Chain/conflux-rust/pull/2438), and there is an English description of the master/backup upgrade operation in this PR. The general steps are as follows:

Assume that the `pos_config/pos_key` file has been obtained

- Prepare two machines, A and B, with `pos_started_as_voter=false` option configured and use the same `pos_config/pos_key`, then let both nodes run. At this time, neither of the nodes will participate in the PoS consensus. 
- Use the command `./conflux rpc local pos voting_status` to get the voting status of the node. If it returns `true`, then the node is voting. If it returns `false`, then it is not voting.
- After confirming that neither node is participating in the PoS, you can use the command `./conflux rpc local pos start_voting` to make one of the nodes start to participate in PoS consensus

At this point, we have two nodes, one master and one backup, and only one node is involved in PoS consensus, so if you need to upgrade a node, you can switch between the primary and backup ones.

- Run command `./conflux rpc local pos stop_voting` to stop the master node from participating in PoS consensus. You can check whether the node is successfully stopped by using the command, `./conflux rpc local pos voting_status`. The node will rename the file `pos_db/secure_storage.json` to `pos_db/secure_storage.json.save` after the voting is stopped.
- Move this file to the `pos_db` directory of the backup node, which will now contain both `pos_db/secure_storage.json` and `pos_db/secure_storage.json.save`.
- Run command `./conflux rpc local pos start_voting` to get the standby node to start participating in PoS consensus, at which point the node will automatically replace the `pos_db/secure_storage.json.save` file with `pos_db/secure_storage.json`. (Note that when performing this step, make sure that the master node is not participating in PoS consensus)

At this point, we have completed the master/backup switchover, and now we can upgrade the nodes that are participating in the consensus operation.

**Again: Make sure to double-check that the master node has stopped participating in the PoS consensus when switching between the master and backup.**

If the master node is down, you can manually rename the node's `pos_db/secure_storage.json` to `pos_db/secure_storage.json.save` and copy it to the backup node's pos_db directory, then let the backup node start participating in the voting.