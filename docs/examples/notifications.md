# Notifications

A wallet calls a listener while it syncs, instead of you polling it. A wallet has to be syncing for the listener to be called: sync it, or start syncing in the background.

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

The daemon has a listener too, for the blocks that are added to the chain, see [Follow new blocks](daemon/blockchain.md#follow-new-blocks).
