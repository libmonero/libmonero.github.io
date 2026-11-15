# Units and networks

## Atomic units

The unit that a program uses is the **atomic unit**, and every amount in the libraries is in it. One XMR is 10<sup>12</sup> atomic units, which is `1000000000000`. Amounts are integers, so that sums are exact: a floating point number cannot hold 0.1 XMR exactly.

Keep amounts in atomic units in your program, and convert to XMR only to show them, or to read what a person typed, with the [utilities](../examples/utilities.md#convert-between-xmr-and-atomic-units). In Java and Kotlin an amount is a `BigInteger`, in C# and C++ a `ulong` or a `uint64_t`, and in Python an `int`.

## Networks

A program runs on one of four networks. Each has its own chain, its own addresses, and its own default ports. An address of one network is not valid on another.

| Network | Use it for | Standard address starts with | Node RPC port |
|---------|-----------|------------------------------|---------------|
| mainnet | real funds | `4` | 18081 |
| stagenet | trying code, with funds that are worth nothing, on a chain that is run like mainnet | `5` | 38081 |
| testnet | testing the changes of Monero itself, and it can be reset | `9` or `A` | 28081 |
| regtest | tests, on a private chain on your machine | the one of mainnet | the one that you give it |

* **Develop on stagenet.** It behaves like mainnet, and funds on it are free. The [examples](../examples/index.md) use it.
* **Test on regtest.** A regtest node mines a block when you ask it to, so you get funds and confirmations in seconds. Use it for automatic tests: see [Generate blocks on regtest](../examples/daemon/mining.md#generate-blocks-on-regtest). It uses the address format of mainnet, and the node says that it is regtest in its [state](../examples/daemon/status.md#get-the-state-of-the-node).
* **Mainnet is real.** Do not point code that is not tested at it.

The network is set when a wallet is created, with its network type: `MoneroNetworkType.STAGENET` in Python, Java, Kotlin and C#, `monero_network_type::STAGENET` in C++. The wallet and the node that it talks to have to be on the same network.

A subaddress has a different first character, on every network: `8` on mainnet, `7` on stagenet, `B` on testnet. Check an address before you use it, with the [utilities](../examples/utilities.md#check-an-address).
