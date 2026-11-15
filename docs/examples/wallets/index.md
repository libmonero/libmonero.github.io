# Wallet types

A wallet comes in a few implementations. They all conform to the same interface, `MoneroWallet`, so the code that reads a balance, lists transactions or creates a transfer is the same for each of them. What changes is where the wallet lives, what it needs, and how you create or open it.

## The implementations

| Implementation | What it is | Use it for |
|----------------|------------|------------|
| [Wallet RPC](rpc.md) | `MoneroWalletRpc`, the client of a `monero-wallet-rpc` server. The server keeps the keys and the files, and syncs the wallet. It has one wallet open at a time. | An application that does not hold the keys itself, or that talks to a wallet that is already running. |
| [Full wallet](full.md) | `MoneroWalletFull`, a wallet in your process, with its files. It syncs with a daemon, and creates and relays transactions. | An application that is the wallet. |
| [Keys wallet](keys.md) | `MoneroWalletKeys`, the keys, the seed and the addresses of an account, and nothing else: no files, no chain, no daemon. | Generating addresses, deriving keys and signing messages, even offline. |

Which language has which implementation:

| | Python | Java | Kotlin | C# | C++ | C |
|-|:------:|:----:|:------:|:--:|:---:|:-:|
| Wallet RPC | yes | yes | yes | yes | yes | yes |
| Full wallet | yes | yes | yes | not yet | yes | yes |
| Keys wallet | yes | not yet | yes | not yet | yes | yes |

On top of the implementation, a wallet can be of a kind that is set by how it is created, with any implementation that can create it:

* A [view-only and an offline wallet](view-only-offline.md) split the keys in two: one wallet sees the funds and builds a transaction, the other one signs it and never goes online.
* A [multisig wallet](multisig.md) needs several participants to sign a transaction.

## One interface

A function that takes a `MoneroWallet` works with every implementation. A function that an implementation does not support throws an error: a keys wallet has no balance, and a wallet on a server cannot do the few things that a server does not offer.

=== "Python"

    ```python
    from monero import *

    def show(wallet: MoneroWallet) -> None:
        print(wallet.get_primary_address())
        print(wallet.get_balance(), wallet.get_unlocked_balance())
        for tx in wallet.get_txs():
            print(tx.hash, tx.is_confirmed)

    show(wallet_rpc)     # a MoneroWalletRpc
    show(wallet_full)    # a MoneroWalletFull
    ```

=== "Java"

    ```java
    void show(MoneroWallet wallet) {
      System.out.println(wallet.getPrimaryAddress());
      System.out.println(wallet.getBalance() + " " + wallet.getUnlockedBalance());
      for (MoneroTxWallet tx : wallet.getTxs()) {
        System.out.println(tx.getHash() + " " + tx.isConfirmed());
      }
    }

    show(walletRpc);     // a MoneroWalletRpc
    show(walletFull);    // a MoneroWalletFull
    ```

=== "Kotlin"

    ```kotlin
    fun show(wallet: MoneroWallet) {
        println(wallet.getPrimaryAddress())
        println("${wallet.getBalance()} ${wallet.getUnlockedBalance()}")
        for (tx in wallet.getTxs()) {
            println("${tx.hash} ${tx.isConfirmed}")
        }
    }

    show(walletRpc)      // a MoneroWalletRpc
    show(walletFull)     // a MoneroWalletFull
    ```

=== "C#"

    ```csharp
    async Task Show(MoneroWallet wallet)
    {
        Console.WriteLine(await wallet.GetPrimaryAddressAsync());
        Console.WriteLine($"{await wallet.GetBalanceAsync()} {await wallet.GetUnlockedBalanceAsync()}");
        foreach (var tx in await wallet.GetTxsAsync())
        {
            Console.WriteLine($"{tx.Hash} {tx.IsConfirmed}");
        }
    }

    await Show(walletRpc);    // a MoneroWalletRpc, the implementation of monero-csharp
    ```

=== "C++"

    ```c++
    void show(monero_wallet& wallet) {
      cout << wallet.get_primary_address() << endl;
      cout << wallet.get_balance() << " " << wallet.get_unlocked_balance() << endl;
      vector<shared_ptr<monero_tx_wallet>> txs = wallet.get_txs();
      for (const shared_ptr<monero_tx_wallet>& tx : txs) {
        cout << tx->m_hash.get() << " " << tx->m_is_confirmed.get() << endl;
      }
      monero_utils::free(txs);
    }

    show(*wallet_rpc);   // a monero_wallet_rpc
    show(*wallet_full);  // a monero_wallet_full
    ```

=== "C"

    ```c
    // every implementation is a monero_wallet*
    void show(monero_wallet* wallet) {
      char* address = NULL;
      monero_wallet_get_primary_address(wallet, &address);
      uint64_t balance = 0, unlocked_balance = 0;
      monero_wallet_get_balance(wallet, &balance);
      monero_wallet_get_unlocked_balance(wallet, &unlocked_balance);
      printf("%s %llu %llu\n", address, (unsigned long long) balance, (unsigned long long) unlocked_balance);
      monero_utils_free(address);
    }

    show(wallet_rpc);    // from monero_wallet_rpc_open()
    show(wallet_full);   // from monero_wallet_create_from_seed()
    ```

## Which one to use

* If you already run `monero-wallet-rpc`, or you want the keys and the files out of your process, use the **wallet RPC**. It is the one that every language has.
* If your application is the wallet, and you want no other server than a daemon, use a **full wallet**.
* If you only need keys, addresses or signatures, use a **keys wallet**: it is the lightest, and it needs no network.
* To keep the spend key on a machine that is never online, use a **view-only and an offline wallet**. To share the spending among several people, use a **multisig wallet**.

Whatever you pick, an amount is in atomic units: 1 XMR is `1000000000000`.
