# Transactions and the pool

Look transactions up, see the pool of the ones that are waiting for a block, and send a signed transaction to the network.

## Look up a transaction

Get a transaction from the node by its hash: one, or several at once. A hash that the node does not know gives no transaction (`None`, `null` or a null pointer).

=== "Python"

    ```python
    tx = daemon.get_tx(tx_hash)                    # None if the node does not know it
    if tx is not None:
        print(tx.fee, tx.is_confirmed, tx.num_confirmations)

    txs = daemon.get_txs([tx_hash_1, tx_hash_2])   # several at once
    ```

=== "Java"

    ```java
    MoneroTx tx = daemon.getTx(txHash);            // null if the node does not know it
    if (tx != null) {
      System.out.println(tx.getFee() + " " + tx.isConfirmed() + " " + tx.getNumConfirmations());
    }

    List<MoneroTx> txs = daemon.getTxs(Arrays.asList(txHash1, txHash2));   // several at once
    ```

=== "Kotlin"

    ```kotlin
    val tx = daemon.getTx(txHash)                  // null if the node does not know it
    if (tx != null) {
        println("${tx.fee} ${tx.isConfirmed} ${tx.numConfirmations}")
    }

    val txs = daemon.getTxs(listOf(txHash1, txHash2))   // several at once
    ```

=== "C#"

    ```csharp
    MoneroTx? tx = await daemon.GetTxAsync(txHash);   // null if the node does not know it
    if (tx != null)
    {
        Console.WriteLine($"{tx.Fee} {tx.IsConfirmed} {tx.NumConfirmations}");
    }

    var txs = await daemon.GetTxsAsync(new[] { txHash1, txHash2 });   // several at once
    ```

=== "C++"

    ```c++
    shared_ptr<monero_tx> tx = daemon->get_tx(tx_hash);   // null if the node does not know it
    if (tx != nullptr) {
      cout << tx->m_fee.get() << " " << tx->m_is_confirmed.get() << endl;
      monero_utils::free(tx);
    }

    vector<shared_ptr<monero_tx>> txs = daemon->get_txs({ tx_hash_1, tx_hash_2 });   // several at once
    for (const shared_ptr<monero_tx>& t : txs) monero_utils::free(t);
    ```

=== "C"

    ```c
    char* tx = NULL;
    monero_daemon_get_tx(daemon, tx_hash, false, &tx);   // JSON, in its block. tx stays NULL if the node does not know it
    if (tx != NULL) printf("%s\n", tx);
    monero_utils_free(tx);

    const char* hashes[] = { tx_hash_1, tx_hash_2 };
    char* txs = NULL;
    monero_daemon_get_txs(daemon, hashes, 2, false, &txs);   // several at once
    monero_utils_free(txs);
    ```

## Get the raw transaction

The hex of a transaction, as the network serializes it. A hash that the node does not know gives nothing.

=== "Python"

    ```python
    tx_hex = daemon.get_tx_hex(tx_hash)               # None if the node does not know it
    tx_hexes = daemon.get_tx_hexes([tx_hash_1, tx_hash_2])   # several at once
    ```

=== "Java"

    ```java
    String txHex = daemon.getTxHex(txHash);           // null if the node does not know it
    List<String> txHexes = daemon.getTxHexes(Arrays.asList(txHash1, txHash2));   // several at once
    ```

=== "Kotlin"

    ```kotlin
    val txHex = daemon.getTxHex(txHash)               // the hex of the transaction
    val txHexes = daemon.getTxHexes(listOf(txHash1, txHash2))   // several at once
    ```

=== "C#"

    ```csharp
    string? txHex = await daemon.GetTxHexAsync(txHash);    // null if the node does not know it
    var txHexes = await daemon.GetTxHexesAsync(new[] { txHash1, txHash2 });   // several at once
    ```

=== "C++"

    ```c++
    boost::optional<string> tx_hex = daemon->get_tx_hex(tx_hash);   // none if the node does not know it
    vector<string> tx_hexes = daemon->get_tx_hexes({ tx_hash_1, tx_hash_2 });   // several at once
    ```

=== "C"

    ```c
    char* tx_hex = NULL;
    monero_daemon_get_tx_hex(daemon, tx_hash, false, &tx_hex);
    monero_utils_free(tx_hex);

    const char* hashes[] = { tx_hash_1, tx_hash_2 };
    char* tx_hexes = NULL;
    monero_daemon_get_tx_hexes(daemon, hashes, 2, false, &tx_hexes);   // JSON array
    monero_utils_free(tx_hexes);
    ```

## Look at the pool

The pool has the transactions that the node has seen and that are not in a block yet. Get their hashes, or the numbers about the pool as a whole. [Read the chain](blockchain.md#read-the-chain) shows how to get the transactions themselves.

=== "Python"

    ```python
    hashes = daemon.get_tx_pool_hashes()

    stats = daemon.get_tx_pool_stats()
    print(stats.num_txs, stats.bytes_total, stats.fee_total)
    print(stats.oldest_timestamp)
    print(stats.num_double_spends, stats.num_failing, stats.num_not_relayed)
    ```

=== "Java"

    ```java
    List<String> hashes = daemon.getTxPoolHashes();

    MoneroTxPoolStats stats = daemon.getTxPoolStats();
    System.out.println(stats.getNumTxs() + " " + stats.getBytesTotal() + " " + stats.getFeeTotal());
    System.out.println(stats.getOldestTimestamp());
    System.out.println(stats.getNumDoubleSpends() + " " + stats.getNumFailing() + " " + stats.getNumNotRelayed());
    ```

=== "Kotlin"

    ```kotlin
    val hashes = daemon.getTxPoolHashes()

    val stats = daemon.getTxPoolStats()
    println("${stats.numTxs} ${stats.bytesTotal} ${stats.feeTotal}")
    println(stats.oldestTimestamp)
    println("${stats.numDoubleSpends} ${stats.numFailing} ${stats.numNotRelayed}")
    ```

=== "C#"

    ```csharp
    var hashes = await daemon.GetTxPoolHashesAsync();

    var stats = await daemon.GetTxPoolStatsAsync();
    Console.WriteLine($"{stats.NumTxs} {stats.BytesTotal} {stats.FeeTotal}");
    Console.WriteLine(stats.OldestTimestamp);
    Console.WriteLine($"{stats.NumDoubleSpends} {stats.NumFailing} {stats.NumNotRelayed}");
    ```

=== "C++"

    ```c++
    vector<string> hashes = daemon->get_tx_pool_hashes();

    shared_ptr<monero_tx_pool_stats> stats = daemon->get_tx_pool_stats();
    cout << stats->m_num_txs.get() << " " << stats->m_bytes_total.get() << " " << stats->m_fee_total.get() << endl;
    cout << stats->m_oldest_timestamp.get() << endl;
    cout << stats->m_num_double_spends.get() << " " << stats->m_num_failing.get() << " " << stats->m_num_not_relayed.get() << endl;
    ```

=== "C"

    ```c
    char* hashes = NULL;
    monero_daemon_get_tx_pool_hashes(daemon, &hashes);   // JSON array
    monero_utils_free(hashes);

    char* stats = NULL;
    monero_daemon_get_tx_pool_stats(daemon, &stats);     // JSON, with the numbers of the pool
    printf("%s\n", stats);
    monero_utils_free(stats);
    ```

## Estimate the fee

The node estimates the fee per byte for a transaction to be mined soon. A wallet uses it to set the fee of a transaction, so you only need it to show or to check one. The result is in atomic units, and `fees` has the estimate of each priority.

=== "Python"

    ```python
    estimate = daemon.get_fee_estimate()

    print(estimate.fee)                  # fee per byte, in atomic units
    print(estimate.fees)                 # one for each priority
    print(estimate.quantization_mask)
    ```

=== "Java"

    ```java
    MoneroFeeEstimate estimate = daemon.getFeeEstimate();

    System.out.println(estimate.getFee());                // fee per byte, in atomic units
    System.out.println(estimate.getFees());               // one for each priority
    System.out.println(estimate.getQuantizationMask());
    ```

=== "Kotlin"

    ```kotlin
    val estimate = daemon.getFeeEstimate()

    println(estimate.fee)                 // fee per byte, in atomic units
    println(estimate.fees)                // one for each priority
    println(estimate.quantizationMask)
    ```

=== "C#"

    ```csharp
    var estimate = await daemon.GetFeeEstimateAsync();

    Console.WriteLine(estimate.Fee);                 // fee per byte, in atomic units
    Console.WriteLine(string.Join(", ", estimate.Fees!));   // one for each priority
    Console.WriteLine(estimate.QuantizationMask);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_fee_estimate> estimate = daemon->get_fee_estimate();

    cout << estimate->m_fee.get() << endl;   // fee per byte, in atomic units
    ```

=== "C"

    ```c
    char* estimate = NULL;
    monero_daemon_get_fee_estimate(daemon, 0, &estimate);   // JSON, with the fee, the fee of each priority and the quantization mask
    printf("%s\n", estimate);
    monero_utils_free(estimate);
    ```

## Submit a transaction

Send a signed transaction to the node, which checks it, adds it to its pool and relays it to the network. `tx_hex` is the hex of a signed transaction, such as the `full_hex` of a transaction that a [wallet](../wallets/index.md) created without relaying it. The result tells if the node accepted it, and if not, why.

=== "Python"

    ```python
    result = daemon.submit_tx_hex(tx_hex)

    if result.is_good:
        print("accepted")
    else:
        print(result.reason, result.is_double_spend, result.is_fee_too_low)
    ```

=== "Java"

    ```java
    MoneroSubmitTxResult result = daemon.submitTxHex(txHex);

    if (result.isGood()) {
      System.out.println("accepted");
    } else {
      System.out.println(result.getReason() + " " + result.isDoubleSpend() + " " + result.isFeeTooLow());
    }
    ```

=== "Kotlin"

    ```kotlin
    val result = daemon.submitTxHex(txHex)

    if (result.isGood == true) {
        println("accepted")
    } else {
        println("${result.reason} ${result.isDoubleSpend} ${result.isFeeTooLow}")
    }
    ```

=== "C#"

    ```csharp
    var result = await daemon.SubmitTxHexAsync(txHex);

    if (result.IsGood == true)
    {
        Console.WriteLine("accepted");
    }
    else
    {
        Console.WriteLine($"{result.Reason} {result.IsDoubleSpend} {result.IsFeeTooLow}");
    }
    ```

=== "C++"

    ```c++
    shared_ptr<monero_submit_tx_result> result = daemon->submit_tx_hex(tx_hex);

    if (result->m_is_good.get()) {
      cout << "accepted" << endl;
    } else {
      cout << result->m_reason.get() << endl;
    }
    ```

=== "C"

    ```c
    char* result = NULL;
    monero_daemon_submit_tx_hex(daemon, tx_hex, false, &result);   // JSON, with isGood and the reason. true as the third argument adds it to the pool without relaying it
    printf("%s\n", result);
    monero_utils_free(result);
    ```

## Relay a transaction again, and flush the pool

Ask the node to relay a transaction of its pool to the network again, for instance when it was not relayed the first time. Flushing the pool removes all its transactions: use it on a test node, never on one that you share.

=== "Python"

    ```python
    daemon.relay_tx_by_hash(tx_hash)     # the transaction has to be in the pool

    daemon.flush_tx_pool()               # removes every transaction of the pool
    ```

=== "Java"

    ```java
    daemon.relayTxByHash(txHash);        // the transaction has to be in the pool

    daemon.flushTxPool();                // removes every transaction of the pool
    ```

=== "Kotlin"

    ```kotlin
    daemon.relayTxByHash(txHash)         // the transaction has to be in the pool

    daemon.flushTxPool()                 // removes every transaction of the pool
    ```

=== "C#"

    ```csharp
    await daemon.RelayTxByHashAsync(txHash);    // the transaction has to be in the pool

    await daemon.FlushTxPoolAsync();            // removes every transaction of the pool
    ```

=== "C++"

    ```c++
    daemon->relay_tx_by_hash(tx_hash);   // the transaction has to be in the pool

    daemon->flush_tx_pool();             // removes every transaction of the pool
    ```

=== "C"

    ```c
    monero_daemon_relay_tx_by_hash(daemon, tx_hash);   // the transaction has to be in the pool

    monero_daemon_flush_tx_pool(daemon);               // removes every transaction of the pool
    ```
