# Examples

Every library has the same data model and API, so the examples read the same in every language: pick a language in any tab and all the pages follow. They run against a stagenet node and a `monero-wallet-rpc` server on your machine.

!!! warning

    Wallets handle real funds. Develop against regtest, stagenet or testnet, and never log or commit a seed or a private key.

## What is in the examples

* [RPC connection](rpc-connection.md): make a connection, check it, connect a wallet to a daemon, and go through Tor.
* [Daemon](daemon/index.md): the calls of the node, by category: [blockchain](daemon/blockchain.md), [status](daemon/status.md), [transactions and the pool](daemon/transactions.md), [outputs and key images](daemon/outputs.md), [peers and network](daemon/network.md), [mining](daemon/mining.md) and [managing the node](daemon/manage.md).
* [Wallets](wallets/index.md): the wallet implementations and how to use each one: [wallet RPC](wallets/rpc.md), [full wallet](wallets/full.md), [keys wallet](wallets/keys.md), [view-only and offline](wallets/view-only-offline.md) and [multisig](wallets/multisig.md).
* [Transactions](transactions.md): send funds and find a transaction.
* [Notifications](notifications.md): be called when a wallet receives funds.

## Get a library

| Language | How to get it |
|----------|---------------|
| Python | Prebuilt `.deb` and wheel packages in the [releases](https://github.com/libmonero/monero-python/releases) |
| Java | Gradle or Maven, or [build it from source](https://github.com/libmonero/monero-java#using-monero-java-in-your-project) |
| Kotlin | [Build it from source](https://github.com/libmonero/monero-kotlin/blob/main/docs/building.md) and publish it with `./gradlew publishToMavenLocal`, it is not in a repository yet |
| C# | `Monero.NET` and `Monero.Native`, [build them from source](https://github.com/libmonero/monero-csharp#using-monero-csharp-in-your-project), they are not on nuget.org yet |
| C++ | [Build it from source](https://github.com/libmonero/monero-cpp#using-monero-cpp-in-your-project) with CMake |
| C | [Build it from source](https://github.com/libmonero/monero-c#building-monero-c-from-source), as a shared or a static library |

## Run a node and a wallet server

The RPC clients need a `monerod` and a `monero-wallet-rpc` to talk to. [docker-monero-rpc](https://github.com/libmonero/docker-monero-rpc) has a compose file that starts both on stagenet:

```bash
git clone https://github.com/libmonero/docker-monero-rpc
cd docker-monero-rpc
docker compose -f examples/docker-compose.yml up -d
```

| Server | URL | Login |
|--------|-----|-------|
| `monerod` | `http://localhost:38081` | `superuser` / `abctesting123` |
| `monero-wallet-rpc` | `http://localhost:38083` | `rpc_user` / `abc123` |

The node has to download the stagenet chain the first time, which takes a while. To try the code without waiting, use the [regtest compose file](https://github.com/libmonero/docker-monero-rpc/blob/main/examples/regtest/docker-compose.yml), and take the ports and the logins from it.

A native wallet, the ones with `Full` and `Keys` in the name, runs in your process and needs no wallet server. A full wallet connects to the daemon, and a keys wallet needs nothing.

## Handle errors

A call that fails throws a `MoneroError` in Python, Java, Kotlin and C#, and a `std::runtime_error` in C++. In C every call that can fail returns `MONERO_OK` or `MONERO_ERROR`, and the message is in `monero_last_error()`. The C examples leave the checks out to stay short.

## Go further

* [Libraries](../libraries.md) has the repository of every language, with its own sample code, build instructions and tests.
* The [data model and API specification](https://github.com/libmonero/monero-python/blob/main/docs/monero-spec.pdf) shows every type and function with diagrams, and applies to every language.
* The [monero-python API reference](https://libmonero.github.io/monero-python/) documents the wallet and daemon interfaces, and its names translate to the other languages.
