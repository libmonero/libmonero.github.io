# Daemon

The daemon clients talk to a `monerod` over its RPC. Every language has them, and they have the same calls: pick a category.

* [Blockchain](blockchain.md): the height, the blocks, the headers, the forks, the hard fork and the new blocks.
* [Daemon status](status.md): the state of the node, its sync, its version, and whether it is trusted.
* [Transactions and the pool](transactions.md): look up transactions, see the pool, estimate the fee and submit a transaction.
* [Outputs and key images](outputs.md): if an output is spent, and the histogram and the distribution of the outputs.
* [Peers and network](network.md): the peers, the bans, and the limits of bandwidth and connections.
* [Mining](mining.md): the mining status, start and stop, the block template and regtest blocks.
* [Manage the node](manage.md): prune, save, set the log level and stop.

The examples expect the node of the [overview](../index.md#run-a-node-and-a-wallet-server) on `localhost:38081`, and a `daemon` made as in [RPC connection](../rpc-connection.md).

## Restricted and unrestricted

A node can serve a restricted RPC, as the public nodes do. It answers the calls that read the chain and the pool, and refuses the ones that change the node or that cost it a lot: mining, banning peers, limits, pruning and stopping. The state of the node tells which one you got: `is_restricted` is true on a restricted node, see [Daemon status](status.md#get-the-state-of-the-node).

A call that a node refuses fails with an error, as any other call that fails. See [Errors](../errors.md).
