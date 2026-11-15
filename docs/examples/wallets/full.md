# Full wallet

A full wallet runs in your process. It has its own files, syncs with a daemon, and creates and relays transactions, so it needs a daemon and no wallet server. The examples expect the node of the [overview](../index.md#run-a-node-and-a-wallet-server) on `localhost:38081`.

!!! note

    Python, Java, Kotlin, C++ and C have a full wallet. monero-csharp has the [wallet RPC](rpc.md) for now, and the other pages of the examples apply to it.

## Create a new wallet

A wallet made without a seed is a random one. The wallet is saved in its files, at the path that you give it and with the password that you give it. Write the seed down: it is the only way to get the wallet back if the files are lost.

=== "Python"

    ```python
    from monero import *

    config = MoneroWalletConfig()
    config.path = "sample_wallet_full"
    config.password = "supersecretpassword123"
    config.network_type = MoneroNetworkType.STAGENET
    config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")

    wallet = MoneroWalletFull.create_wallet(config)

    print(wallet.get_seed())               # write it down, and keep it safe
    print(wallet.get_primary_address())
    ```

=== "Java"

    ```java
    import monero.daemon.model.MoneroNetworkType;
    import monero.wallet.MoneroWalletFull;
    import monero.wallet.model.MoneroWalletConfig;

    MoneroWalletFull wallet = MoneroWalletFull.createWallet(new MoneroWalletConfig()
            .setPath("sample_wallet_full")
            .setPassword("supersecretpassword123")
            .setNetworkType(MoneroNetworkType.STAGENET)
            .setServerUri("http://localhost:38081")
            .setServerUsername("superuser")
            .setServerPassword("abctesting123"));

    System.out.println(wallet.getSeed());  // write it down, and keep it safe
    System.out.println(wallet.getPrimaryAddress());
    ```

=== "Kotlin"

    ```kotlin
    import monero.common.MoneroRpcConnection
    import monero.daemon.model.MoneroNetworkType
    import monero.wallet.MoneroWalletFull
    import monero.wallet.model.MoneroWalletConfig

    val config = MoneroWalletConfig()
    config.path = "sample_wallet_full"
    config.password = "supersecretpassword123"
    config.networkType = MoneroNetworkType.STAGENET
    config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")

    MoneroWalletFull.createWallet(config).use { wallet ->
        println(wallet.getSeed())          // write it down, and keep it safe
        println(wallet.getPrimaryAddress())
    }
    ```

=== "C++"

    ```c++
    using namespace monero;

    monero_wallet_config config;
    config.m_path = "sample_wallet_full";
    config.m_password = "supersecretpassword123";
    config.m_network_type = monero_network_type::STAGENET;
    config.m_server = make_shared<monero_rpc_connection>("http://localhost:38081", "superuser", "abctesting123");

    monero_wallet* wallet = monero_wallet_full::create_wallet(config);

    cout << wallet->get_seed() << endl;    // write it down, and keep it safe
    cout << wallet->get_primary_address() << endl;

    delete wallet;
    ```

=== "C"

    ```c
    #include "monero_c.h"

    monero_wallet* wallet = NULL;
    monero_wallet_create_random("sample_wallet_full", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET, "English", &wallet);
    monero_wallet_set_daemon_connection(wallet, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);

    char* seed = NULL;
    monero_wallet_get_seed(wallet, &seed); // write it down, and keep it safe
    monero_utils_free(seed);

    monero_wallet_free(wallet);
    ```

## Restore a wallet from a seed and sync it

A native wallet is made of files in your process, and it syncs with a daemon by itself. Give it the daemon, the seed and a restore height, the height of the chain where it starts looking for transactions, and sync it. The listener receives the progress.

=== "Python"

    ```python
    from monero import *

    class SyncListener(MoneroWalletListener):
        def on_sync_progress(self, height: int, start_height: int, end_height: int, percent_done: float, message: str) -> None:
            print(f"{percent_done * 100:.1f}% {message}")

    config = MoneroWalletConfig()
    config.path = "sample_wallet_full"
    config.password = "supersecretpassword123"
    config.network_type = MoneroNetworkType.STAGENET
    config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    config.seed = "hefty value scenic..."
    config.restore_height = 573936

    wallet = MoneroWalletFull.create_wallet(config)
    wallet.sync(SyncListener())  # blocks until the wallet is synced
    wallet.start_syncing(5000)   # then keep it synced in the background, every 5 seconds

    wallet.close(True)  # save it and close it
    ```

=== "Java"

    ```java
    import monero.daemon.model.MoneroNetworkType;
    import monero.wallet.MoneroWalletFull;
    import monero.wallet.model.MoneroWalletConfig;
    import monero.wallet.model.MoneroWalletListener;

    MoneroWalletFull wallet = MoneroWalletFull.createWallet(new MoneroWalletConfig()
            .setPath("sample_wallet_full")
            .setPassword("supersecretpassword123")
            .setNetworkType(MoneroNetworkType.STAGENET)
            .setServerUri("http://localhost:38081")
            .setServerUsername("superuser")
            .setServerPassword("abctesting123")
            .setSeed("hefty value scenic...")
            .setRestoreHeight(573936l));

    wallet.sync(new MoneroWalletListener() {  // blocks until the wallet is synced
      @Override
      public void onSyncProgress(long height, long startHeight, long endHeight, double percentDone, String message) {
        System.out.printf("%.1f%% %s%n", percentDone * 100, message);
      }
    });
    wallet.startSyncing(5000l);  // then keep it synced in the background, every 5 seconds

    wallet.close(true);  // save it and close it
    ```

=== "Kotlin"

    ```kotlin
    import monero.common.MoneroRpcConnection
    import monero.daemon.model.MoneroNetworkType
    import monero.wallet.MoneroWalletFull
    import monero.wallet.model.MoneroWalletConfig
    import monero.wallet.model.MoneroWalletListener

    val config = MoneroWalletConfig()
    config.path = "sample_wallet_full"
    config.password = "supersecretpassword123"
    config.networkType = MoneroNetworkType.STAGENET
    config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    config.seed = "hefty value scenic..."
    config.restoreHeight = 573936

    MoneroWalletFull.createWallet(config).use { wallet ->
        wallet.sync(object : MoneroWalletListener() {  // blocks until the wallet is synced
            override fun onSyncProgress(height: Long, startHeight: Long, endHeight: Long, percentDone: Double, message: String?) {
                println("${percentDone * 100}% $message")
            }
        })
        wallet.startSyncing(5000)  // then keep it synced in the background, every 5 seconds

        wallet.save()  // `use` closes it without saving
    }
    ```

=== "C#"

    ```csharp
    using Monero.Wallet;
    using Monero.Wallet.Model;

    // monero-csharp has the wallet RPC client for now: the server keeps the wallet and syncs it
    await using var wallet = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");
    await wallet.OpenWalletAsync("sample_wallet_rpc", "supersecretpassword123");

    await wallet.SyncAsync(listener: new SyncListener());  // waits until the wallet is synced
    await wallet.StartSyncingAsync(5000);                  // then keep it synced in the background, every 5 seconds

    await wallet.CloseAsync(true);  // save it and close it

    class SyncListener : MoneroWalletListener
    {
        public override void OnSyncProgress(ulong height, ulong startHeight, ulong endHeight, double percentDone, string message)
        {
            Console.WriteLine($"{percentDone * 100:F1}% {message}");
        }
    }
    ```

=== "C++"

    ```c++
    using namespace monero;

    monero_wallet_config config;
    config.m_path = "sample_wallet_full";
    config.m_password = "supersecretpassword123";
    config.m_network_type = monero_network_type::STAGENET;
    config.m_server = make_shared<monero_rpc_connection>("http://localhost:38081", "superuser", "abctesting123");
    config.m_seed = "hefty value scenic...";
    config.m_restore_height = 573936;
    monero_wallet* wallet = monero_wallet_full::create_wallet(config);

    struct : monero_wallet_listener {
      void on_sync_progress(uint64_t height, uint64_t start_height, uint64_t end_height, double percent_done, const string& message) {
        cout << percent_done * 100 << "% " << message << endl;
      }
    } sync_listener;
    wallet->sync(sync_listener);  // blocks until the wallet is synced
    wallet->start_syncing(5000);  // then keep it synced in the background, every 5 seconds

    wallet->close(true);  // save it and close it
    delete wallet;
    ```

=== "C"

    ```c
    #include <stdio.h>
    #include "monero_c.h"

    static void on_sync_progress(void* user_data, uint64_t height, uint64_t start_height, uint64_t end_height, double percent_done, const char* message) {
      printf("%.1f%% %s\n", percent_done * 100, message);
    }

    monero_wallet* wallet = NULL;
    monero_wallet_create_from_seed("sample_wallet_full", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET,
                                   "hefty value scenic...", "", 573936, NULL, &wallet);
    monero_wallet_set_daemon_connection(wallet, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, on_sync_progress, NULL, NULL, NULL, NULL};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);

    char* result = NULL;
    monero_wallet_sync(wallet, NULL, listener, &result);  // blocks until the wallet is synced
    monero_utils_free(result);
    monero_wallet_start_syncing(wallet, 5000);  // then keep it synced in the background, every 5 seconds

    monero_wallet_close(wallet, true);  // save it and close it
    monero_wallet_free(wallet);
    monero_wallet_listener_free(listener);
    ```

The files of the wallet do not keep the connection to the daemon. When you open a wallet again, set its daemon connection before you sync it.

## Open a wallet

Open the wallet from its files. The files do not keep the connection to the daemon, so set it before you sync.

=== "Python"

    ```python
    if MoneroWalletFull.wallet_exists("sample_wallet_full"):
        wallet = MoneroWalletFull.open_wallet("sample_wallet_full", "supersecretpassword123", MoneroNetworkType.STAGENET)
        wallet.set_daemon_connection("http://localhost:38081", "superuser", "abctesting123")
        wallet.sync()
    ```

=== "Java"

    ```java
    if (MoneroWalletFull.walletExists("sample_wallet_full")) {
      MoneroWalletFull wallet = MoneroWalletFull.openWallet("sample_wallet_full", "supersecretpassword123", MoneroNetworkType.STAGENET,
              new MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123"));
      wallet.sync();
    }
    ```

=== "Kotlin"

    ```kotlin
    if (MoneroWalletFull.walletExists("sample_wallet_full")) {
        val config = MoneroWalletConfig()
        config.path = "sample_wallet_full"
        config.password = "supersecretpassword123"
        config.networkType = MoneroNetworkType.STAGENET

        MoneroWalletFull.openWallet(config).use { wallet ->
            wallet.setDaemonConnection("http://localhost:38081", "superuser", "abctesting123")
            wallet.sync()
        }
    }
    ```

=== "C++"

    ```c++
    if (monero_wallet_full::wallet_exists("sample_wallet_full")) {
      monero_wallet_full* wallet = monero_wallet_full::open_wallet("sample_wallet_full", "supersecretpassword123", monero_network_type::STAGENET);
      wallet->set_daemon_connection("http://localhost:38081", "superuser", "abctesting123");
      wallet->sync();
      delete wallet;
    }
    ```

=== "C"

    ```c
    bool exists = false;
    monero_wallet_exists("sample_wallet_full", &exists);
    if (exists) {
      monero_wallet* wallet = NULL;
      monero_wallet_open("sample_wallet_full", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET, &wallet);
      monero_wallet_set_daemon_connection(wallet, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);

      char* result = NULL;
      monero_wallet_sync(wallet, NULL, NULL, &result);
      monero_utils_free(result);

      monero_wallet_free(wallet);
    }
    ```

## Save and close

A full wallet writes its files when you save it. Closing it without saving leaves the files as they were at the last save, and the next sync catches up from there.

=== "Python"

    ```python
    wallet.save()
    wallet.close()          # or wallet.close(True) to save and close in one call
    ```

=== "Java"

    ```java
    wallet.save();
    wallet.close();         // or wallet.close(true) to save and close in one call
    ```

=== "Kotlin"

    ```kotlin
    wallet.save()
    wallet.close(false)     // `use` closes it too, without saving
    ```

=== "C++"

    ```c++
    wallet->save();
    wallet->close();        // or wallet->close(true) to save and close in one call
    delete wallet;
    ```

=== "C"

    ```c
    monero_wallet_save(wallet);
    monero_wallet_close(wallet, false);   // or true to save and close in one call
    monero_wallet_free(wallet);
    ```
