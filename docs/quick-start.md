# Quick start

Connect to a daemon and read the chain height, in every language. Each example expects a `monerod` RPC server on `localhost:38081`, for instance one of the images of [docker-monero-rpc](https://github.com/libmonero/docker-monero-rpc).

=== "Python"

    ```python
    from monero import *

    daemon = MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123")
    height = daemon.get_height()                    # 1523651

    wallet = MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123")
    wallet.open_wallet("sample_wallet_rpc", "supersecretpassword123")
    balance = wallet.get_balance()                  # 533648366742 (atomic units)
    ```

=== "Java"

    ```java
    import monero.daemon.MoneroDaemon;
    import monero.daemon.MoneroDaemonRpc;

    MoneroDaemon daemon = new MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123");
    long height = daemon.getHeight();               // 1523651
    ```

=== "Kotlin"

    ```kotlin
    import monero.daemon.MoneroDaemonRpc

    MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123").use { daemon ->
        val height = daemon.getHeight()             // 1523651
    }
    ```

=== "C#"

    ```csharp
    using Monero.Daemon;

    MoneroDaemon daemon = new MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123");
    ulong height = await daemon.GetHeightAsync();   // 1523651
    ```

The wallets work the same way: an RPC wallet and a full wallet with native bindings implement the same `MoneroWallet` interface. The README of each library has a complete sample, with a wallet created from a seed, sync listeners and a transfer.

!!! warning

    Wallets handle real funds. Develop against regtest, stagenet or testnet, and never log or commit a seed or a private key.
