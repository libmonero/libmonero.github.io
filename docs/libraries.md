# Libraries

## Language libraries

| Repository | Language | Bindings | Notes |
|------------|----------|----------|-------|
| [**monero-cpp**](https://github.com/libmonero/monero-cpp) | C++ | wraps `wallet2` | Core library and reference data model. Wallet and daemon RPC clients, full, keys-only, multisig, view-only and offline wallets. |
| [**monero-c**](https://github.com/libmonero/monero-c) | C | C ABI over monero-cpp | Opaque handles, JSON data and listener callbacks. Foundation for FFI bindings. |
| [**monero-python**](https://github.com/libmonero/monero-python) | Python 3.10 to 3.14 | pybind11 | Prebuilt `.deb` and wheel packages, [documentation](https://libmonero.github.io/monero-python/). |
| [**monero-java**](https://github.com/libmonero/monero-java) | Java | JNI | Over 300 tests. |
| [**monero-kotlin**](https://github.com/libmonero/monero-kotlin) | Kotlin | monero-c | Linux, macOS, Windows and Android (AAR). |
| [**monero-csharp**](https://github.com/libmonero/monero-csharp) | C# (.NET 10) | monero-c (P/Invoke) | Async API with cancellation. |

## Tooling

| Repository | Description |
|------------|-------------|
| [**docker-monero-rpc**](https://github.com/libmonero/docker-monero-rpc) | Docker images for `monerod` and `monero-wallet-rpc`, for development and testing against regtest or mainnet. |

## Documentation

* [monero-python API reference](https://libmonero.github.io/monero-python/)
* [Data model and API specification](https://github.com/libmonero/monero-python/blob/main/docs/monero-spec.pdf)

!!! note

    Most of these projects are under development and unfunded. Each repository states its own status at the top of its README.
