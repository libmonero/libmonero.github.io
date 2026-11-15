# Sync and balances

A wallet knows what it owns by reading the chain, block by block, from its restore height. That is the **sync**. Until it has synced, the balance and the transactions of a wallet are the ones that it knew when it last stopped. This page shows how to sync a wallet, how far it is, and what to do when it looks wrong.

The calls are the ones of every wallet implementation. A [wallet on a server](rpc.md) is synced by its server, and it answers these calls too. A [keys wallet](keys.md) never syncs.

## Sync a wallet

`sync` reads the chain up to the top, and returns when the wallet has caught up. The result has the number of blocks that it read and whether it found funds. Give it a height to start from, to read the chain from there and not from where the wallet stopped.

=== "Python"

    ```python
    result = wallet.sync()
    print(result.num_blocks_fetched, result.received_money)

    result = wallet.sync(573936)           # from this height
    ```

=== "Java"

    ```java
    MoneroSyncResult result = wallet.sync();
    System.out.println(result.getNumBlocksFetched() + " " + result.getReceivedMoney());

    result = wallet.sync(573936l);         // from this height
    ```

=== "Kotlin"

    ```kotlin
    var result = wallet.sync()
    println("${result.numBlocksFetched} ${result.receivedMoney}")

    result = wallet.sync(573936)           // from this height
    ```

=== "C#"

    ```csharp
    var result = await wallet.SyncAsync();
    Console.WriteLine($"{result.NumBlocksFetched} {result.ReceivedMoney}");

    result = await wallet.SyncAsync(573936);    // from this height
    ```

=== "C++"

    ```c++
    monero_sync_result result = wallet->sync();
    cout << result.m_num_blocks_fetched.get() << " " << result.m_received_money.get() << endl;

    result = wallet->sync(573936);         // from this height
    ```

=== "C"

    ```c
    char* result = NULL;
    monero_wallet_sync(wallet, NULL, NULL, &result);       // JSON, with the blocks that it read and whether it found funds
    printf("%s\n", result);
    monero_utils_free(result);

    uint64_t start_height = 573936;
    monero_wallet_sync(wallet, &start_height, NULL, &result);   // from this height
    monero_utils_free(result);
    ```

To see the progress of a long sync, give `sync` a listener: see [Notifications](../notifications.md#follow-the-progress-of-a-sync).

## Sync in the background

A wallet can keep itself synced, polling the daemon at the period that you give it, in milliseconds. A listener is called as funds come in, see [Notifications](../notifications.md).

=== "Python"

    ```python
    wallet.start_syncing(5000)      # every 5 seconds

    wallet.stop_syncing()
    ```

=== "Java"

    ```java
    wallet.startSyncing(5000l);     // every 5 seconds

    wallet.stopSyncing();
    ```

=== "Kotlin"

    ```kotlin
    wallet.startSyncing(5000)       // every 5 seconds

    wallet.stopSyncing()
    ```

=== "C#"

    ```csharp
    await wallet.StartSyncingAsync(5000);    // every 5 seconds

    await wallet.StopSyncingAsync();
    ```

=== "C++"

    ```c++
    wallet->start_syncing(5000);    // every 5 seconds

    wallet->stop_syncing();
    ```

=== "C"

    ```c
    monero_wallet_start_syncing(wallet, 5000);    // every 5 seconds

    monero_wallet_stop_syncing(wallet);
    ```

## Know how far the wallet is

The height of the wallet is the last block that it read, and the height of the daemon is the top of the chain that the daemon knows. The wallet is synced when they are the same.

=== "Python"

    ```python
    print(wallet.get_height(), wallet.get_daemon_height())
    print(wallet.is_synced())
    print(wallet.is_connected_to_daemon())
    ```

=== "Java"

    ```java
    long height = wallet.getHeight();
    long daemonHeight = wallet.getDaemonHeight();

    System.out.println(height + " " + daemonHeight);
    System.out.println(height >= daemonHeight);           // synced
    System.out.println(wallet.isConnectedToDaemon());
    ```

=== "Kotlin"

    ```kotlin
    println("${wallet.getHeight()} ${wallet.getDaemonHeight()}")
    println(wallet.isSynced())
    println(wallet.isConnectedToDaemon())
    ```

=== "C#"

    ```csharp
    Console.WriteLine($"{await wallet.GetHeightAsync()} {await wallet.GetDaemonHeightAsync()}");
    Console.WriteLine(await wallet.IsSyncedAsync());
    Console.WriteLine(await wallet.IsConnectedToDaemonAsync());
    ```

=== "C++"

    ```c++
    cout << wallet->get_height() << " " << wallet->get_daemon_height() << endl;
    cout << wallet->is_synced() << endl;
    cout << wallet->is_connected_to_daemon() << endl;
    ```

=== "C"

    ```c
    uint64_t height = 0, daemon_height = 0;
    monero_wallet_get_height(wallet, &height);
    monero_wallet_get_daemon_height(wallet, &daemon_height);

    bool synced = false;
    monero_wallet_is_synced(wallet, &synced);

    bool connected = false;
    monero_wallet_is_connected_to_daemon(wallet, &connected);
    ```

## Find the height of a date

A new wallet does not need to look at the blocks from before it existed. To restore a wallet, give it a restore height, and use the height of the day it was created, or a day before to be safe. The wallet finds it from a date.

=== "Python"

    ```python
    restore_height = wallet.get_height_by_date(2024, 3, 15)    # year, month, day

    config = MoneroWalletConfig()
    config.seed = "hefty value scenic..."
    config.restore_height = restore_height
    ```

=== "Java"

    ```java
    long restoreHeight = wallet.getHeightByDate(2024, 3, 15);    // year, month, day

    MoneroWalletConfig config = new MoneroWalletConfig()
            .setSeed("hefty value scenic...")
            .setRestoreHeight(restoreHeight);
    ```

=== "Kotlin"

    ```kotlin
    val restoreHeight = wallet.getHeightByDate(2024, 3, 15)    // year, month, day

    val config = MoneroWalletConfig()
    config.seed = "hefty value scenic..."
    config.restoreHeight = restoreHeight
    ```

=== "C#"

    ```csharp
    ulong restoreHeight = await wallet.GetHeightByDateAsync(2024, 3, 15);    // year, month, day

    var config = new MoneroWalletConfig
    {
        Seed = "hefty value scenic...",
        RestoreHeight = restoreHeight
    };
    ```

=== "C++"

    ```c++
    uint64_t restore_height = wallet->get_height_by_date(2024, 3, 15);    // year, month, day

    monero_wallet_config config;
    config.m_seed = "hefty value scenic...";
    config.m_restore_height = restore_height;
    ```

=== "C"

    ```c
    uint64_t restore_height = 0;
    monero_wallet_get_height_by_date(wallet, 2024, 3, 15, &restore_height);    // year, month, day
    ```

The call asks the daemon, so it needs a wallet that is connected to one. Another wallet that is connected to the same network can answer it for the wallet that you are about to restore.

## Read the balances

The balance is what the wallet holds, and the unlocked balance is the part of it that it can spend now. Funds that arrived in the last 10 blocks, and the change of a transaction that you sent, are locked. Ask for the whole wallet, for an account, or for a subaddress.

=== "Python"

    ```python
    wallet.get_balance(), wallet.get_unlocked_balance()           # the wallet
    wallet.get_balance(0), wallet.get_unlocked_balance(0)         # account 0
    wallet.get_balance(0, 1), wallet.get_unlocked_balance(0, 1)   # account 0, subaddress 1
    ```

=== "Java"

    ```java
    wallet.getBalance(); wallet.getUnlockedBalance();             // the wallet
    wallet.getBalance(0); wallet.getUnlockedBalance(0);           // account 0
    wallet.getBalance(0, 1); wallet.getUnlockedBalance(0, 1);     // account 0, subaddress 1
    ```

=== "Kotlin"

    ```kotlin
    wallet.getBalance(); wallet.getUnlockedBalance()              // the wallet
    wallet.getBalance(0); wallet.getUnlockedBalance(0)            // account 0
    wallet.getBalance(0, 1); wallet.getUnlockedBalance(0, 1)      // account 0, subaddress 1
    ```

=== "C#"

    ```csharp
    await wallet.GetBalanceAsync(); await wallet.GetUnlockedBalanceAsync();             // the wallet
    await wallet.GetBalanceAsync(0); await wallet.GetUnlockedBalanceAsync(0);           // account 0
    await wallet.GetBalanceAsync(0, 1); await wallet.GetUnlockedBalanceAsync(0, 1);     // account 0, subaddress 1
    ```

=== "C++"

    ```c++
    wallet->get_balance(); wallet->get_unlocked_balance();             // the wallet
    wallet->get_balance(0); wallet->get_unlocked_balance(0);           // account 0
    wallet->get_balance(0, 1); wallet->get_unlocked_balance(0, 1);     // account 0, subaddress 1
    ```

=== "C"

    ```c
    uint64_t balance = 0, unlocked_balance = 0;
    monero_wallet_get_balance(wallet, &balance);                         // the wallet
    monero_wallet_get_unlocked_balance(wallet, &unlocked_balance);
    monero_wallet_get_account_balance(wallet, 0, &balance);              // account 0
    monero_wallet_get_subaddress_balance(wallet, 0, 1, &balance);        // account 0, subaddress 1
    ```

## Fix a wallet that looks wrong

Two calls for a wallet whose balance or whose list of transactions is not what you expect. Try the light one first.

* **Rescan the spent outputs** asks the daemon again which of the outputs of the wallet are spent. It fixes a wallet that shows as unspent an output that is spent.
* **Rescan the blockchain** throws away what the wallet knows of the chain, and syncs it again from the restore height. What the chain cannot give back, such as the notes on transactions, is lost. It takes as long as the first sync, and the wallet must be connected to a daemon while it runs.

=== "Python"

    ```python
    wallet.rescan_spent()

    wallet.rescan_blockchain()
    ```

=== "Java"

    ```java
    wallet.rescanSpent();

    wallet.rescanBlockchain();
    ```

=== "Kotlin"

    ```kotlin
    wallet.rescanSpent()

    wallet.rescanBlockchain()
    ```

=== "C#"

    ```csharp
    await wallet.RescanSpentAsync();

    await wallet.RescanBlockchainAsync();
    ```

=== "C++"

    ```c++
    wallet->rescan_spent();

    wallet->rescan_blockchain();
    ```

=== "C"

    ```c
    monero_wallet_rescan_spent(wallet);

    monero_wallet_rescan_blockchain(wallet);
    ```

To have the wallet look at one transaction that it missed, give its hash to `scan_txs`:

=== "Python"

    ```python
    wallet.scan_txs([tx_hash])
    ```

=== "Java"

    ```java
    wallet.scanTxs(Arrays.asList(txHash));
    ```

=== "Kotlin"

    ```kotlin
    wallet.scanTxs(listOf(txHash))
    ```

=== "C#"

    ```csharp
    await wallet.ScanTxsAsync(new[] { txHash });
    ```

=== "C++"

    ```c++
    wallet->scan_txs({ tx_hash });
    ```

=== "C"

    ```c
    const char* hashes[] = { tx_hash };
    monero_wallet_scan_txs(wallet, hashes, 1);
    ```
