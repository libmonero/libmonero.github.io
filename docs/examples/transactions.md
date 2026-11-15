# Transactions

Create, relay and find transactions. The examples use a `wallet` of any [type](wallets/index.md), with funds, and an `address` to send to: on stagenet, use an address of that network.

## Send funds

Create the transaction without relaying it, show the fee to the user, and relay it when they agree. The same code works for every wallet type: a wallet on a wallet RPC server and a native wallet create and relay a transaction the same way.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000  # 0.25 XMR, in atomic units
    tx_config.relay = False          # create it, do not relay it yet

    tx = wallet.create_tx(tx_config)
    fee = tx.fee  # "Are you sure you want to send... ?"

    wallet.relay_tx(tx)  # relay it
    ```

=== "Java"

    ```java
    MoneroTxWallet tx = wallet.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000")  // 0.25 XMR, in atomic units
            .setRelay(false));          // create it, do not relay it yet

    BigInteger fee = tx.getFee();  // "Are you sure you want to send... ?"

    wallet.relayTx(tx);  // relay it
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, amount))  // amount in atomic units
    txConfig.relay = false                                              // create it, do not relay it yet

    val tx = wallet.createTxs(txConfig).first()
    val fee = tx.fee  // "Are you sure you want to send... ?"

    val hash = wallet.relayTx(tx)  // relay it
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000,  // 0.25 XMR, in atomic units
        Relay = false           // create it, do not relay it yet
    };

    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    ulong? fee = tx.Fee;  // "Are you sure you want to send... ?"

    await wallet.RelayTxAsync(tx);  // relay it
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;  // 0.25 XMR, in atomic units
    tx_config.m_relay = false;          // create it, do not relay it yet

    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(tx_config);
    uint64_t fee = tx->m_fee.get();  // "Are you sure you want to send... ?"

    wallet->relay_tx(*tx);  // relay it
    monero_utils::free(tx);
    ```

=== "C"

    ```c
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}],\"relay\":false}", address);

    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);  // JSON, "Are you sure you want to send... ?"

    char* hash = NULL;
    monero_wallet_relay_tx_json(wallet, tx, &hash);  // relay it

    monero_utils_free(tx);
    monero_utils_free(hash);
    ```

A transaction takes about 10 blocks to unlock its outputs, so the change of a transfer is locked for a while and the unlocked balance is lower than the balance until then.

## Find a transaction by hash

Ask the wallet for the transactions that match a query. A query can match on more than the hash: it has a field for most of the properties of a transaction.

=== "Python"

    ```python
    query = MoneroTxQuery()
    query.hashes = [tx_hash]

    txs = wallet.get_txs(query)
    print(txs[0].fee, txs[0].is_confirmed)
    ```

=== "Java"

    ```java
    List<MoneroTxWallet> txs = wallet.getTxs(new MoneroTxQuery().setHashes(txHash));

    System.out.println(txs.get(0).getFee() + " " + txs.get(0).isConfirmed());
    ```

=== "Kotlin"

    ```kotlin
    val query = MoneroTxQuery()
    query.hashes = listOf(txHash)

    val txs = wallet.getTxs(query)
    println("${txs.first().fee} ${txs.first().isConfirmed}")
    ```

=== "C#"

    ```csharp
    var txs = await wallet.GetTxsAsync(new MoneroTxQuery { Hashes = new List<string> { txHash } });

    Console.WriteLine($"{txs[0].Fee} {txs[0].IsConfirmed}");
    ```

=== "C++"

    ```c++
    monero_tx_query query;
    query.m_hashes = { tx_hash };

    vector<shared_ptr<monero_tx_wallet>> txs = wallet->get_txs(query);
    cout << txs[0]->m_fee.get() << " " << txs[0]->m_is_confirmed.get() << endl;
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char query[256];
    snprintf(query, sizeof(query), "{\"hashes\":[\"%s\"]}", tx_hash);

    char* txs = NULL;
    monero_wallet_get_txs(wallet, query, &txs);   // JSON, the transactions in their blocks
    monero_utils_free(txs);
    ```

A transaction that the wallet does not have is not in the result.
