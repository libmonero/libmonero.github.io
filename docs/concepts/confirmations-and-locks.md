# Confirmations and locks

A payment is not final when it is sent. It goes through steps, and funds are not always spendable at once. The libraries tell you where a transaction is, and how much of a balance can be spent.

## Where a transaction is

1. **Created.** A wallet has made the transaction. Nothing has left the wallet yet, and you can still throw it away.
2. **Relayed.** The wallet has sent it to a node, which puts it in its **pool** and passes it on. It is not in the chain yet. The `in_tx_pool` flag of the transaction is true.
3. **Confirmed.** A miner has put it in a block, and the transaction has one **confirmation**. Each block that is added on top adds a confirmation. The number is `num_confirmations`.
4. **Failed.** A transaction that is not valid anymore, such as one that spends an output that another transaction spent first, is marked as failed.

A block comes about every two minutes. A transaction that is in the pool is not safe to rely on, since it can be dropped. A transaction with a few confirmations is safe for most uses, and one with 10 is safe for almost all of them, since the reorganizations of the chain are short.

See [Look up a transaction](../examples/daemon/transactions.md#look-up-a-transaction) and [Find transactions](../examples/transactions/find.md).

## Locked funds

Funds that a wallet receives are **locked** for a while, and are not part of the unlocked balance:

* An output that a wallet receives is locked until 10 blocks have passed since its transaction was mined.
* The **change** of a transaction that you send comes back to your wallet as a new output, and it is locked for the same 10 blocks. So after a transfer, the unlocked balance is lower than the balance, for about 20 minutes.
* The reward of a mined block, the coinbase, is locked for 60 blocks.
* A transaction can ask for a longer lock, with its unlock time. It is a field of the transaction.

The **balance** is what the wallet owns, and the **unlocked balance** is what it can spend now. A transfer that needs more than the unlocked balance fails. See [Read the balances](../examples/wallets/sync.md#read-the-balances), and, for a subaddress, the number of blocks that its funds wait, `num_blocks_to_unlock`, in [Look at the subaddresses of an account](../examples/wallets/accounts.md#look-at-the-subaddresses-of-an-account).

## Waiting for a payment

To know when a payment arrives, you do not need to poll: [add a listener](../examples/notifications.md). It is called when the wallet receives the funds, and when they unlock. A listener for the [new blocks](../examples/notifications.md#follow-the-new-blocks-of-a-wallet) lets you count the confirmations.

