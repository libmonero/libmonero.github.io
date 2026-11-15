# Notifications

A wallet calls a listener while it syncs, instead of you polling it. A wallet has to be syncing for the listener to be called: [sync it](wallets/sync.md), or start syncing in the background. A listener is a class that you write, with a method for each event that you care about. The others do nothing.

| Event | Called when |
|-------|-------------|
| sync progress | a sync reads blocks, with how far it is |
| new block | the wallet has read a new block |
| output received | the wallet receives funds, when they are confirmed, and when they unlock |
| output spent | the wallet spends an output |
| balances changed | the balance or the unlocked balance changes |

The daemon has a listener too, for the blocks that are added to the chain: see [Follow new blocks](daemon/blockchain.md#follow-new-blocks).

## Follow the progress of a sync

Give `sync` a listener and it reports its progress as it reads the blocks, with a percentage and a message. It is what a progress bar shows.

=== "Python"

    ```python
    from monero import *

    class SyncListener(MoneroWalletListener):
        def on_sync_progress(self, height: int, start_height: int, end_height: int, percent_done: float, message: str) -> None:
            print(f"{percent_done * 100:.1f}% {message}")

    wallet.sync(SyncListener())
    ```

=== "Java"

    ```java
    wallet.sync(new MoneroWalletListener() {
      @Override
      public void onSyncProgress(long height, long startHeight, long endHeight, double percentDone, String message) {
        System.out.printf("%.1f%% %s%n", percentDone * 100, message);
      }
    });
    ```

=== "Kotlin"

    ```kotlin
    wallet.sync(object : MoneroWalletListener() {
        override fun onSyncProgress(height: Long, startHeight: Long, endHeight: Long, percentDone: Double, message: String?) {
            println("${percentDone * 100}% $message")
        }
    })
    ```

=== "C#"

    ```csharp
    await wallet.SyncAsync(listener: new SyncListener());

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
    struct : monero_wallet_listener {
      void on_sync_progress(uint64_t height, uint64_t start_height, uint64_t end_height, double percent_done, const string& message) {
        cout << percent_done * 100 << "% " << message << endl;
      }
    } sync_listener;

    wallet->sync(sync_listener);
    ```

=== "C"

    ```c
    static void on_sync_progress(void* user_data, uint64_t height, uint64_t start_height, uint64_t end_height, double percent_done, const char* message) {
      printf("%.1f%% %s\n", percent_done * 100, message);
    }

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, on_sync_progress, NULL, NULL, NULL, NULL};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);

    char* result = NULL;
    monero_wallet_sync(wallet, NULL, listener, &result);
    monero_utils_free(result);
    ```

## Get notified of funds

Add a listener to a wallet and it is called when the wallet receives funds, when they are confirmed, and when they are unlocked and can be spent, while the wallet syncs.

=== "Python"

    ```python
    class FundsListener(MoneroWalletListener):
        def on_output_received(self, output: MoneroOutputWallet) -> None:
            amount = output.amount  # atomic units
            tx_hash = output.tx.hash
            is_confirmed = output.tx.is_confirmed
            is_locked = output.tx.is_locked

        def on_balances_changed(self, new_balance: int, new_unlocked_balance: int) -> None:
            print(new_balance, new_unlocked_balance)

    wallet.add_listener(FundsListener())
    wallet.start_syncing(5000)
    ```

=== "Java"

    ```java
    wallet.addListener(new MoneroWalletListener() {
      @Override
      public void onOutputReceived(MoneroOutputWallet output) {
        BigInteger amount = output.getAmount();  // atomic units
        String txHash = output.getTx().getHash();
        Boolean isConfirmed = output.getTx().isConfirmed();
        Boolean isLocked = output.getTx().isLocked();
      }

      @Override
      public void onBalancesChanged(BigInteger newBalance, BigInteger newUnlockedBalance) {
        System.out.println(newBalance + " " + newUnlockedBalance);
      }
    });
    wallet.startSyncing(5000l);
    ```

=== "Kotlin"

    ```kotlin
    wallet.addListener(object : MoneroWalletListener() {
        override fun onOutputReceived(output: MoneroOutputWallet) {
            val amount = output.amount  // atomic units
            val txHash = output.tx?.hash
            val isConfirmed = output.tx?.isConfirmed
            val isLocked = output.tx?.isLocked
        }

        override fun onBalancesChanged(newBalance: BigInteger, newUnlockedBalance: BigInteger) {
            println("$newBalance $newUnlockedBalance")
        }
    })
    wallet.startSyncing(5000)
    ```

=== "C#"

    ```csharp
    wallet.AddListener(new FundsListener());
    await wallet.StartSyncingAsync(5000);

    class FundsListener : MoneroWalletListener
    {
        public override void OnOutputReceived(MoneroOutputWallet output)
        {
            ulong? amount = output.Amount;  // atomic units
        }

        public override void OnBalancesChanged(ulong newBalance, ulong newUnlockedBalance)
        {
            Console.WriteLine($"{newBalance} {newUnlockedBalance}");
        }
    }
    ```

=== "C++"

    ```c++
    struct : monero_wallet_listener {
      void on_output_received(const monero_output_wallet& output) {
        uint64_t amount = output.m_amount.get();  // atomic units
        string tx_hash = output.m_tx->m_hash.get();
        bool is_confirmed = output.m_tx->m_is_confirmed.get();
        bool is_locked = dynamic_pointer_cast<monero_tx_wallet>(output.m_tx)->m_is_locked.get();
      }
    } funds_listener;
    wallet->add_listener(funds_listener);
    wallet->start_syncing(5000);
    ```

=== "C"

    ```c
    static void on_output_received(void* user_data, const char* output_json) {
      // JSON of a monero_output_wallet, with the amount and the transaction
    }

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, NULL, NULL, NULL, on_output_received, NULL};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);
    monero_wallet_add_listener(wallet, listener);
    monero_wallet_start_syncing(wallet, 5000);
    ```

Keep the wallet open and synced for as long as you want the notifications.

## Know when funds are spent

The wallet is told when one of its outputs is spent, by a transaction that it made or by one made with the same keys somewhere else. The output has its amount and the transaction that spent it.

=== "Python"

    ```python
    class SpentListener(MoneroWalletListener):
        def on_output_spent(self, output: MoneroOutputWallet) -> None:
            print(output.amount, output.tx.hash)

    wallet.add_listener(SpentListener())
    ```

=== "Java"

    ```java
    wallet.addListener(new MoneroWalletListener() {
      @Override
      public void onOutputSpent(MoneroOutputWallet output) {
        System.out.println(output.getAmount() + " " + output.getTx().getHash());
      }
    });
    ```

=== "Kotlin"

    ```kotlin
    wallet.addListener(object : MoneroWalletListener() {
        override fun onOutputSpent(output: MoneroOutputWallet) {
            println("${output.amount} ${output.tx?.hash}")
        }
    })
    ```

=== "C#"

    ```csharp
    wallet.AddListener(new SpentListener());

    class SpentListener : MoneroWalletListener
    {
        public override void OnOutputSpent(MoneroOutputWallet output)
        {
            Console.WriteLine($"{output.Amount} {output.Tx?.Hash}");
        }
    }
    ```

=== "C++"

    ```c++
    struct : monero_wallet_listener {
      void on_output_spent(const monero_output_wallet& output) {
        cout << output.m_amount.get() << " " << output.m_tx->m_hash.get() << endl;
      }
    } spent_listener;

    wallet->add_listener(spent_listener);
    ```

=== "C"

    ```c
    static void on_output_spent(void* user_data, const char* output_json) {
      // JSON of a monero_output_wallet, with the amount and the transaction that spent it
    }

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, NULL, NULL, NULL, NULL, on_output_spent};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);
    monero_wallet_add_listener(wallet, listener);
    ```

## Follow the balance

The listener is called with the new balance and the new unlocked balance each time that they change. It is easier than adding up the funds that were received and spent.

=== "Python"

    ```python
    class BalanceListener(MoneroWalletListener):
        def on_balances_changed(self, new_balance: int, new_unlocked_balance: int) -> None:
            print(new_balance, new_unlocked_balance)

    wallet.add_listener(BalanceListener())
    ```

=== "Java"

    ```java
    wallet.addListener(new MoneroWalletListener() {
      @Override
      public void onBalancesChanged(BigInteger newBalance, BigInteger newUnlockedBalance) {
        System.out.println(newBalance + " " + newUnlockedBalance);
      }
    });
    ```

=== "Kotlin"

    ```kotlin
    wallet.addListener(object : MoneroWalletListener() {
        override fun onBalancesChanged(newBalance: BigInteger, newUnlockedBalance: BigInteger) {
            println("$newBalance $newUnlockedBalance")
        }
    })
    ```

=== "C#"

    ```csharp
    wallet.AddListener(new BalanceListener());

    class BalanceListener : MoneroWalletListener
    {
        public override void OnBalancesChanged(ulong newBalance, ulong newUnlockedBalance)
        {
            Console.WriteLine($"{newBalance} {newUnlockedBalance}");
        }
    }
    ```

=== "C++"

    ```c++
    struct : monero_wallet_listener {
      void on_balances_changed(uint64_t new_balance, uint64_t new_unlocked_balance) {
        cout << new_balance << " " << new_unlocked_balance << endl;
      }
    } balance_listener;

    wallet->add_listener(balance_listener);
    ```

=== "C"

    ```c
    static void on_balances_changed(void* user_data, uint64_t new_balance, uint64_t new_unlocked_balance) {
      printf("%llu %llu\n", (unsigned long long) new_balance, (unsigned long long) new_unlocked_balance);
    }

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, NULL, NULL, on_balances_changed, NULL, NULL};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);
    monero_wallet_add_listener(wallet, listener);
    ```

## Follow the new blocks of a wallet

The listener is called with the height of each new block that the wallet reads. It tells you that the wallet moves, which is useful to wait for confirmations: the number of confirmations of a transaction grows with each block.

=== "Python"

    ```python
    class BlockListener(MoneroWalletListener):
        def on_new_block(self, height: int) -> None:
            print("block", height)

    wallet.add_listener(BlockListener())
    ```

=== "Java"

    ```java
    wallet.addListener(new MoneroWalletListener() {
      @Override
      public void onNewBlock(long height) {
        System.out.println("block " + height);
      }
    });
    ```

=== "Kotlin"

    ```kotlin
    wallet.addListener(object : MoneroWalletListener() {
        override fun onNewBlock(height: Long) {
            println("block $height")
        }
    })
    ```

=== "C#"

    ```csharp
    wallet.AddListener(new BlockListener());

    class BlockListener : MoneroWalletListener
    {
        public override void OnNewBlock(ulong height)
        {
            Console.WriteLine($"block {height}");
        }
    }
    ```

=== "C++"

    ```c++
    struct : monero_wallet_listener {
      void on_new_block(uint64_t height) {
        cout << "block " << height << endl;
      }
    } block_listener;

    wallet->add_listener(block_listener);
    ```

=== "C"

    ```c
    static void on_new_block(void* user_data, uint64_t height) {
      printf("block %llu\n", (unsigned long long) height);
    }

    monero_wallet_listener_callbacks callbacks = {sizeof(monero_wallet_listener_callbacks), NULL, NULL, on_new_block, NULL, NULL, NULL};
    monero_wallet_listener* listener = NULL;
    monero_wallet_listener_create(&callbacks, &listener);
    monero_wallet_add_listener(wallet, listener);
    ```

## Stop listening

Remove a listener that you do not need any more. A wallet keeps calling the listeners that were added to it, so remove one before you free it.

=== "Python"

    ```python
    listener = BalanceListener()
    wallet.add_listener(listener)

    wallet.remove_listener(listener)
    ```

=== "Java"

    ```java
    MoneroWalletListener listener = new MoneroWalletListener() { /* ... */ };
    wallet.addListener(listener);

    wallet.removeListener(listener);
    ```

=== "Kotlin"

    ```kotlin
    val listener = object : MoneroWalletListener() { /* ... */ }
    wallet.addListener(listener)

    wallet.removeListener(listener)
    ```

=== "C#"

    ```csharp
    var listener = new BalanceListener();
    wallet.AddListener(listener);

    wallet.RemoveListener(listener);
    ```

=== "C++"

    ```c++
    wallet->add_listener(balance_listener);

    wallet->remove_listener(balance_listener);
    ```

=== "C"

    ```c
    monero_wallet_add_listener(wallet, listener);

    monero_wallet_remove_listener(wallet, listener);
    monero_wallet_listener_free(listener);
    ```
