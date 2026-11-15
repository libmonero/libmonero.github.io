# Wallet RPC

A `monero-wallet-rpc` server keeps the keys, the files and the sync, and has one wallet open at a time. The client is the one implementation that every language has, and it has the functions of a full wallet, except the few that a server does not offer.

The examples expect the server of the [overview](../index.md#run-a-node-and-a-wallet-server) on `localhost:38083`. Until you create or open a wallet on it, the server has none, and a call that needs a wallet fails.

## Create a wallet on the server

Create a wallet once, and [open it](#open-a-wallet) afterwards. The wallet is a file in the directory of the server.

=== "Python"

    ```python
    from monero import *

    wallet = MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123")

    config = MoneroWalletConfig()
    config.path = "sample_wallet_rpc"
    config.password = "supersecretpassword123"
    wallet.create_wallet(config)                    # a random wallet

    print(wallet.get_seed())                        # write it down, and keep it safe
    print(wallet.get_primary_address())
    ```

=== "Java"

    ```java
    import monero.wallet.MoneroWalletRpc;
    import monero.wallet.model.MoneroWalletConfig;

    MoneroWalletRpc wallet = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");

    wallet.createWallet(new MoneroWalletConfig()    // a random wallet
            .setPath("sample_wallet_rpc")
            .setPassword("supersecretpassword123"));

    System.out.println(wallet.getSeed());           // write it down, and keep it safe
    System.out.println(wallet.getPrimaryAddress());
    ```

=== "Kotlin"

    ```kotlin
    import monero.wallet.MoneroWalletRpc
    import monero.wallet.model.MoneroWalletConfig

    MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123").use { wallet ->
        val config = MoneroWalletConfig()
        config.path = "sample_wallet_rpc"
        config.password = "supersecretpassword123"
        wallet.createWallet(config)                 // a random wallet

        println(wallet.getSeed())                   // write it down, and keep it safe
        println(wallet.getPrimaryAddress())
    }
    ```

=== "C#"

    ```csharp
    using Monero.Wallet;
    using Monero.Wallet.Model;

    await using var wallet = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");

    await wallet.CreateWalletAsync(new MoneroWalletConfig   // a random wallet
    {
        Path = "sample_wallet_rpc",
        Password = "supersecretpassword123"
    });

    Console.WriteLine(await wallet.GetSeedAsync());         // write it down, and keep it safe
    Console.WriteLine(await wallet.GetPrimaryAddressAsync());
    ```

=== "C++"

    ```c++
    using namespace monero;

    monero_wallet_rpc* wallet = new monero_wallet_rpc("http://localhost:38083", "rpc_user", "abc123");

    shared_ptr<monero_wallet_config> config = make_shared<monero_wallet_config>();
    config->m_path = "sample_wallet_rpc";
    config->m_password = "supersecretpassword123";
    wallet->create_wallet(config);                  // a random wallet

    cout << wallet->get_seed() << endl;             // write it down, and keep it safe
    cout << wallet->get_primary_address() << endl;

    delete wallet;
    ```

=== "C"

    ```c
    #include "monero_c.h"

    monero_wallet* wallet = NULL;
    monero_wallet_rpc_create_random("http://localhost:38083", "rpc_user", "abc123",
                                    "sample_wallet_rpc", "supersecretpassword123", "English", &wallet);

    char* seed = NULL;
    monero_wallet_get_seed(wallet, &seed);          // write it down, and keep it safe
    monero_utils_free(seed);

    monero_wallet_free(wallet);
    ```

## Restore a wallet from a seed

Give the server the seed, and the height where it starts to look for transactions. A lower height finds older transactions and takes longer.

=== "Python"

    ```python
    config = MoneroWalletConfig()
    config.path = "restored_wallet_rpc"
    config.password = "supersecretpassword123"
    config.seed = "hefty value scenic..."
    config.restore_height = 573936
    wallet.create_wallet(config)
    ```

=== "Java"

    ```java
    wallet.createWallet(new MoneroWalletConfig()
            .setPath("restored_wallet_rpc")
            .setPassword("supersecretpassword123")
            .setSeed("hefty value scenic...")
            .setRestoreHeight(573936l));
    ```

=== "Kotlin"

    ```kotlin
    val config = MoneroWalletConfig()
    config.path = "restored_wallet_rpc"
    config.password = "supersecretpassword123"
    config.seed = "hefty value scenic..."
    config.restoreHeight = 573936
    wallet.createWallet(config)
    ```

=== "C#"

    ```csharp
    await wallet.CreateWalletAsync(new MoneroWalletConfig
    {
        Path = "restored_wallet_rpc",
        Password = "supersecretpassword123",
        Seed = "hefty value scenic...",
        RestoreHeight = 573936
    });
    ```

=== "C++"

    ```c++
    shared_ptr<monero_wallet_config> config = make_shared<monero_wallet_config>();
    config->m_path = "restored_wallet_rpc";
    config->m_password = "supersecretpassword123";
    config->m_seed = "hefty value scenic...";
    config->m_restore_height = 573936;
    wallet->create_wallet(config);
    ```

=== "C"

    ```c
    monero_wallet* wallet = NULL;
    monero_wallet_rpc_create_from_seed("http://localhost:38083", "rpc_user", "abc123",
                                       "restored_wallet_rpc", "supersecretpassword123",
                                       "hefty value scenic...", "", 573936, "English", &wallet);
    ```

## Open a wallet

=== "Python"

    ```python
    wallet = MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123")
    wallet.open_wallet("sample_wallet_rpc", "supersecretpassword123")
    ```

=== "Java"

    ```java
    MoneroWalletRpc wallet = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");
    wallet.openWallet("sample_wallet_rpc", "supersecretpassword123");
    ```

=== "Kotlin"

    ```kotlin
    MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123").use { wallet ->
        wallet.openWallet("sample_wallet_rpc", "supersecretpassword123")
    }
    ```

=== "C#"

    ```csharp
    await using var wallet = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");
    await wallet.OpenWalletAsync("sample_wallet_rpc", "supersecretpassword123");
    ```

=== "C++"

    ```c++
    monero_wallet_rpc* wallet = new monero_wallet_rpc("http://localhost:38083", "rpc_user", "abc123");
    wallet->open_wallet("sample_wallet_rpc", "supersecretpassword123");
    ```

=== "C"

    ```c
    monero_wallet* wallet = NULL;
    monero_wallet_rpc_open("http://localhost:38083", "rpc_user", "abc123",
                           "sample_wallet_rpc", "supersecretpassword123", &wallet);
    ```

## Read the wallet

The calls are the ones of every wallet. The server syncs the wallet by itself.

=== "Python"

    ```python
    primary_address = wallet.get_primary_address()  # 555zgduFhmKd2o8rPUz...
    balance = wallet.get_balance()                  # 533648366742, atomic units
    unlocked_balance = wallet.get_unlocked_balance()
    subaddress = wallet.get_address(0, 1)           # account 0, subaddress 1
    txs = wallet.get_txs()                          # transactions to and from the wallet
    ```

=== "Java"

    ```java
    String primaryAddress = wallet.getPrimaryAddress();   // 555zgduFhmKd2o8rPUz...
    BigInteger balance = wallet.getBalance();             // 533648366742, atomic units
    BigInteger unlockedBalance = wallet.getUnlockedBalance();
    String subaddress = wallet.getAddress(0, 1);          // account 0, subaddress 1
    List<MoneroTxWallet> txs = wallet.getTxs();           // transactions to and from the wallet
    ```

=== "Kotlin"

    ```kotlin
    val primaryAddress = wallet.getPrimaryAddress()   // 555zgduFhmKd2o8rPUz...
    val balance = wallet.getBalance()                 // 533648366742, atomic units
    val unlockedBalance = wallet.getUnlockedBalance()
    val subaddress = wallet.getAddress(0, 1)          // account 0, subaddress 1
    val txs = wallet.getTxs()                         // transactions to and from the wallet
    ```

=== "C#"

    ```csharp
    string primaryAddress = await wallet.GetPrimaryAddressAsync();   // 555zgduFhmKd2o8rPUz...
    ulong balance = await wallet.GetBalanceAsync();                  // 533648366742, atomic units
    ulong unlockedBalance = await wallet.GetUnlockedBalanceAsync();
    string subaddress = await wallet.GetAddressAsync(0, 1);          // account 0, subaddress 1
    var txs = await wallet.GetTxsAsync();                            // transactions to and from the wallet
    ```

=== "C++"

    ```c++
    string primary_address = wallet->get_primary_address();       // 555zgduFhmKd2o8rPUz...
    uint64_t balance = wallet->get_balance();                     // 533648366742, atomic units
    uint64_t unlocked_balance = wallet->get_unlocked_balance();
    string subaddress = wallet->get_address(0, 1);                // account 0, subaddress 1
    vector<shared_ptr<monero_tx_wallet>> txs = wallet->get_txs(); // transactions to and from the wallet
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char* primary_address = NULL;
    monero_wallet_get_primary_address(wallet, &primary_address);   // 555zgduFhmKd2o8rPUz...
    uint64_t balance = 0;
    monero_wallet_get_balance(wallet, &balance);                   // 533648366742, atomic units
    char* txs = NULL;
    monero_wallet_get_txs(wallet, NULL, &txs);                     // JSON, transactions to and from the wallet

    monero_utils_free(txs);
    monero_utils_free(primary_address);
    ```

## Close the wallet

Save and close the wallet on the server, so that it can open another one. A server has one wallet open at a time, so a program that needs two wallets, as in a [multisig](multisig.md) setup, uses two servers on two ports.

=== "Python"

    ```python
    wallet.close(True)          # True saves it
    ```

=== "Java"

    ```java
    wallet.close(true);         // true saves it
    ```

=== "Kotlin"

    ```kotlin
    wallet.close(true)          // true saves it. `use` closes it too, but without saving
    ```

=== "C#"

    ```csharp
    await wallet.CloseAsync(true);    // true saves it
    ```

=== "C++"

    ```c++
    wallet->close(true);        // true saves it
    delete wallet;
    ```

=== "C"

    ```c
    monero_wallet_close(wallet, true);   // true saves it
    monero_wallet_free(wallet);
    ```
