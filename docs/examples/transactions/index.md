# Transactions

A transaction of a wallet goes through the same steps whatever the wallet is: it is **created** from a config that says where the funds go, how fast, and from where. The wallet gives back what it will cost. It is **relayed** when you are sure, and the network takes it from there. Later, you **find** it, annotate it, and prove it if you have to.

* [Send funds](send.md): create and relay a transfer, to one or several recipients, with a priority, a split, a fee taken out of the amount, or a payment request.
* [Outputs and sweeping](outputs.md): list the outputs, sweep all the funds, one output or the dust, and freeze an output.
* [Find transactions](find.md): by hash or by properties, what a transaction moved, and the transfers of a subaddress.
* [Notes and proofs](notes-proofs.md): notes on transactions, and the proofs of a payment, a spend and a reserve.

An amount is in atomic units: 1 XMR is `1000000000000`. A transaction that a wallet received is locked for 10 blocks, and the wallet cannot spend its change until then: the unlocked balance is lower than the balance in the meantime.

The calls of the node on transactions, such as the pool and the raw transaction, are in the [daemon examples](../daemon/transactions.md). To be told when funds arrive, see [Notifications](../notifications.md).
