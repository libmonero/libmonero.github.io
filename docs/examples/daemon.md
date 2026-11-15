# Daemon

The daemon clients read the chain from a `monerod`. The examples expect the node of the [overview](index.md#run-a-node-and-a-wallet-server) on `localhost:38081`.

## Read the chain

Connect to the daemon, get the height, the transactions in the pool and the last block.

=== "Python"

    ```python
    from monero import *

    daemon = MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123")

    height = daemon.get_height()        # 1523651
    txs_in_pool = daemon.get_tx_pool()  # transactions waiting to be mined

    block = daemon.get_block_by_height(height - 1)  # the last block
    print(block.hash, block.tx_hashes)
    ```

=== "Java"

    ```java
    import monero.daemon.MoneroDaemon;
    import monero.daemon.MoneroDaemonRpc;
    import monero.daemon.model.MoneroBlock;
    import monero.daemon.model.MoneroTx;

    MoneroDaemon daemon = new MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123");

    long height = daemon.getHeight();               // 1523651
    List<MoneroTx> txsInPool = daemon.getTxPool();  // transactions waiting to be mined

    MoneroBlock block = daemon.getBlockByHeight(height - 1); // the last block
    System.out.println(block.getHash() + " " + block.getTxHashes());
    ```

=== "Kotlin"

    ```kotlin
    import monero.daemon.MoneroDaemonRpc

    MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123").use { daemon ->
        val height = daemon.getHeight()     // 1523651
        val txsInPool = daemon.getTxPool()  // transactions waiting to be mined

        val block = daemon.getBlockByHeight(height - 1)  // the last block
        println("${block.hash} ${block.txHashes}")
    }
    ```

=== "C#"

    ```csharp
    using Monero.Daemon;

    MoneroDaemon daemon = new MoneroDaemonRpc("http://localhost:38081", "superuser", "abctesting123");

    ulong height = await daemon.GetHeightAsync();   // 1523651
    var txsInPool = await daemon.GetTxPoolAsync();  // transactions waiting to be mined

    var block = await daemon.GetBlockByHeightAsync(height - 1); // the last block
    Console.WriteLine($"{block.Hash} {block.TxHashes?.Count}");
    ```

=== "C++"

    ```c++
    using namespace monero;

    monero_daemon* daemon = new monero_daemon_rpc("http://localhost:38081", "superuser", "abctesting123");

    uint64_t height = daemon->get_height();  // 1523651
    vector<shared_ptr<monero_tx>> txs_in_pool = daemon->get_tx_pool(); // transactions waiting to be mined
    for (const shared_ptr<monero_tx>& tx : txs_in_pool) monero_utils::free(tx);

    shared_ptr<monero_block> block = daemon->get_block_by_height(height - 1); // the last block
    cout << block->m_hash.get() << endl;
    monero_utils::free(block);

    delete daemon;
    ```

=== "C"

    ```c
    #include <stdio.h>
    #include "monero_c.h"

    monero_daemon* daemon = NULL;
    monero_daemon_connect("http://localhost:38081", "superuser", "abctesting123", "", "", 0, &daemon);

    uint64_t height = 0;
    monero_daemon_get_height(daemon, &height);  // 1523651

    char* txs_in_pool = NULL;
    monero_daemon_get_tx_pool(daemon, &txs_in_pool);  // JSON, transactions waiting to be mined
    monero_utils_free(txs_in_pool);

    char* block = NULL;
    monero_daemon_get_block_by_height(daemon, height - 1, &block); // JSON, the last block
    printf("%s\n", block);
    monero_utils_free(block);

    monero_daemon_free(daemon);
    ```

## Get the state of the node

The version, the network, the height of the node and the height it is syncing to. A node is ready when `is_synchronized` is true.

=== "Python"

    ```python
    info = daemon.get_info()

    print(info.version, info.network_type)
    print(info.height, info.target_height, info.is_synchronized)
    ```

=== "Java"

    ```java
    MoneroDaemonInfo info = daemon.getInfo();

    System.out.println(info.getVersion() + " " + info.getNetworkType());
    System.out.println(info.getHeight() + " " + info.getTargetHeight() + " " + info.isSynchronized());
    ```

=== "Kotlin"

    ```kotlin
    val info = daemon.getInfo()

    println("${info.version} ${info.networkType}")
    println("${info.height} ${info.targetHeight} ${info.isSynchronized}")
    ```

=== "C#"

    ```csharp
    var info = await daemon.GetInfoAsync();

    Console.WriteLine($"{info.Version} {info.NetworkType}");
    Console.WriteLine($"{info.Height} {info.TargetHeight} {info.IsSynchronized}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_info> info = daemon->get_info();

    cout << info->m_version.get() << endl;
    cout << info->m_height.get() << " " << info->m_target_height.get() << " " << info->m_is_synchronized.get() << endl;
    ```

=== "C"

    ```c
    char* info = NULL;
    monero_daemon_get_info(daemon, &info);   // JSON, with the version, the network type, the heights and is_synchronized
    printf("%s\n", info);
    monero_utils_free(info);
    ```

## Follow new blocks

Wait for the next block, or add a listener that is called for each block that is added to the chain.

=== "Python"

    ```python
    header = daemon.wait_for_next_block_header()   # blocks until the next block
    print(header.height, header.hash)

    class BlockListener(MoneroDaemonListener):
        def on_block_header(self, header: MoneroBlockHeader) -> None:
            print(header.height, header.hash)

    daemon.add_listener(BlockListener())
    ```

=== "Java"

    ```java
    MoneroBlockHeader header = daemon.waitForNextBlockHeader();   // blocks until the next block
    System.out.println(header.getHeight() + " " + header.getHash());

    daemon.addListener(new MoneroDaemonListener() {
      @Override
      public void onBlockHeader(MoneroBlockHeader header) {
        System.out.println(header.getHeight() + " " + header.getHash());
      }
    });
    ```

=== "Kotlin"

    ```kotlin
    val header = daemon.waitForNextBlockHeader()   // blocks until the next block
    println("${header.height} ${header.hash}")

    daemon.addListener(object : MoneroDaemonListener() {
        override fun onBlockHeader(header: MoneroBlockHeader) {
            println("${header.height} ${header.hash}")
        }
    })
    ```

=== "C#"

    ```csharp
    var header = await daemon.WaitForNextBlockHeaderAsync();   // waits until the next block
    Console.WriteLine($"{header.Height} {header.Hash}");

    daemon.AddListener(new BlockListener());

    class BlockListener : MoneroDaemonListener
    {
        public override void OnBlockHeader(MoneroBlockHeader header)
        {
            Console.WriteLine($"{header.Height} {header.Hash}");
        }
    }
    ```

=== "C++"

    ```c++
    shared_ptr<monero_block_header> header = daemon->wait_for_next_block_header();   // blocks until the next block
    cout << header->m_height.get() << " " << header->m_hash.get() << endl;

    struct : monero_daemon_listener {
      void on_block_header(const shared_ptr<monero_block_header>& header) {
        cout << header->m_height.get() << " " << header->m_hash.get() << endl;
      }
    } block_listener;
    daemon->add_listener(block_listener);
    ```

=== "C"

    ```c
    char* header = NULL;
    monero_daemon_wait_for_next_block_header(daemon, &header);   // JSON, blocks until the next block
    monero_utils_free(header);

    static void on_block_header(void* user_data, const char* header_json) {
      printf("%s\n", header_json);
    }

    monero_daemon_listener_callbacks callbacks = {sizeof(monero_daemon_listener_callbacks), NULL, on_block_header};
    monero_daemon_listener* listener = NULL;
    monero_daemon_listener_create(&callbacks, &listener);
    monero_daemon_add_listener(daemon, listener);

    // ...and when you are done
    monero_daemon_remove_listener(daemon, listener);
    monero_daemon_listener_free(listener);
    ```


The same kind of notification exists for a wallet, see [Notifications](notifications.md).

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

## Get block headers

A header has the height, the hash, the time, the reward and the number of transactions of a block, without the transactions. It is lighter than a block, so use it to scan a range of the chain.

=== "Python"

    ```python
    last = daemon.get_last_block_header()
    print(last.height, last.hash, last.timestamp, last.reward)

    headers = daemon.get_block_headers_by_range(last.height - 9, last.height)   # the last 10 blocks
    for header in headers:
        print(header.height, header.num_txs)
    ```

=== "Java"

    ```java
    MoneroBlockHeader last = daemon.getLastBlockHeader();
    System.out.println(last.getHeight() + " " + last.getHash() + " " + last.getTimestamp() + " " + last.getReward());

    List<MoneroBlockHeader> headers = daemon.getBlockHeadersByRange(last.getHeight() - 9, last.getHeight());   // the last 10 blocks
    for (MoneroBlockHeader header : headers) {
      System.out.println(header.getHeight() + " " + header.getNumTxs());
    }
    ```

=== "Kotlin"

    ```kotlin
    val last = daemon.getLastBlockHeader()
    println("${last.height} ${last.hash} ${last.timestamp} ${last.reward}")

    val headers = daemon.getBlockHeadersByRange(last.height!! - 9, last.height!!)   // the last 10 blocks
    for (header in headers) {
        println("${header.height} ${header.numTxs}")
    }
    ```

=== "C#"

    ```csharp
    var last = await daemon.GetLastBlockHeaderAsync();
    Console.WriteLine($"{last.Height} {last.Hash} {last.Timestamp} {last.Reward}");

    var headers = await daemon.GetBlockHeadersByRangeAsync(last.Height!.Value - 9, last.Height!.Value);   // the last 10 blocks
    foreach (var header in headers)
    {
        Console.WriteLine($"{header.Height} {header.NumTxs}");
    }
    ```

=== "C++"

    ```c++
    shared_ptr<monero_block_header> last = daemon->get_last_block_header();
    cout << last->m_height.get() << " " << last->m_hash.get() << " " << last->m_timestamp.get() << endl;

    uint64_t height = last->m_height.get();
    vector<shared_ptr<monero_block_header>> headers = daemon->get_block_headers_by_range(height - 9, height);   // the last 10 blocks
    for (const shared_ptr<monero_block_header>& header : headers) {
      cout << header->m_height.get() << " " << header->m_num_txs.get() << endl;
    }
    ```

=== "C"

    ```c
    char* last = NULL;
    monero_daemon_get_last_block_header(daemon, &last);   // JSON
    printf("%s\n", last);
    monero_utils_free(last);

    char* headers = NULL;
    monero_daemon_get_block_headers_by_range(daemon, height - 9, height, &headers);   // JSON, the last 10 blocks
    monero_utils_free(headers);
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

## Check if an output is spent

The key image of an output tells if it was spent: the node knows the key images of its chain and of its pool, without knowing whose they are. Get the key image of an output from a wallet.

=== "Python"

    ```python
    status = daemon.get_key_image_spent_status(key_image)

    if status == MoneroKeyImageSpentStatus.NOT_SPENT:
        print("not spent")
    elif status == MoneroKeyImageSpentStatus.CONFIRMED:
        print("spent in the chain")
    else:                                     # TX_POOL
        print("spent in a transaction of the pool")

    statuses = daemon.get_key_image_spent_statuses([key_image_1, key_image_2])   # several at once
    ```

=== "Java"

    ```java
    MoneroKeyImageSpentStatus status = daemon.getKeyImageSpentStatus(keyImage);

    if (status == MoneroKeyImageSpentStatus.NOT_SPENT) System.out.println("not spent");
    else if (status == MoneroKeyImageSpentStatus.CONFIRMED) System.out.println("spent in the chain");
    else System.out.println("spent in a transaction of the pool");   // TX_POOL

    List<MoneroKeyImageSpentStatus> statuses = daemon.getKeyImageSpentStatuses(Arrays.asList(keyImage1, keyImage2));   // several at once
    ```

=== "Kotlin"

    ```kotlin
    when (daemon.getKeyImageSpentStatus(keyImage)) {
        MoneroKeyImageSpentStatus.NOT_SPENT -> println("not spent")
        MoneroKeyImageSpentStatus.CONFIRMED -> println("spent in the chain")
        MoneroKeyImageSpentStatus.TX_POOL -> println("spent in a transaction of the pool")
    }

    val statuses = daemon.getKeyImageSpentStatuses(listOf(keyImage1, keyImage2))   // several at once
    ```

=== "C#"

    ```csharp
    var status = await daemon.GetKeyImageSpentStatusAsync(keyImage);

    switch (status)
    {
        case MoneroKeyImageSpentStatus.NotSpent: Console.WriteLine("not spent"); break;
        case MoneroKeyImageSpentStatus.Confirmed: Console.WriteLine("spent in the chain"); break;
        case MoneroKeyImageSpentStatus.TxPool: Console.WriteLine("spent in a transaction of the pool"); break;
    }

    var statuses = await daemon.GetKeyImageSpentStatusesAsync(new[] { keyImage1, keyImage2 });   // several at once
    ```

=== "C++"

    ```c++
    monero_key_image_spent_status status = daemon->get_key_image_spent_status(key_image);

    if (status == monero_key_image_spent_status::NOT_SPENT) cout << "not spent" << endl;
    else if (status == monero_key_image_spent_status::CONFIRMED) cout << "spent in the chain" << endl;
    else cout << "spent in a transaction of the pool" << endl;   // TX_POOL

    vector<monero_key_image_spent_status> statuses = daemon->get_key_image_spent_statuses({ key_image_1, key_image_2 });   // several at once
    ```

=== "C"

    ```c
    monero_key_image_spent_status status;
    monero_daemon_get_key_image_spent_status(daemon, key_image, &status);

    if (status == MONERO_KEY_IMAGE_NOT_SPENT) printf("not spent\n");
    else if (status == MONERO_KEY_IMAGE_CONFIRMED) printf("spent in the chain\n");
    else printf("spent in a transaction of the pool\n");   // MONERO_KEY_IMAGE_TX_POOL
    ```

## See the peers and the sync state

How many peers the node has, and how far it is from the top of the chain while it syncs.

=== "Python"

    ```python
    sync_info = daemon.get_sync_info()
    print(sync_info.height, sync_info.target_height)

    for peer in daemon.get_peers():
        print(peer.host, peer.port, peer.height, peer.is_incoming)
    ```

=== "Java"

    ```java
    MoneroDaemonSyncInfo syncInfo = daemon.getSyncInfo();
    System.out.println(syncInfo.getHeight() + " " + syncInfo.getTargetHeight());

    for (MoneroPeer peer : daemon.getPeers()) {
      System.out.println(peer.getHost() + " " + peer.getPort() + " " + peer.getHeight() + " " + peer.isIncoming());
    }
    ```

=== "Kotlin"

    ```kotlin
    val syncInfo = daemon.getSyncInfo()
    println("${syncInfo.height} ${syncInfo.targetHeight}")

    for (peer in daemon.getPeers()) {
        println("${peer.host} ${peer.port} ${peer.height} ${peer.isIncoming}")
    }
    ```

=== "C#"

    ```csharp
    var syncInfo = await daemon.GetSyncInfoAsync();
    Console.WriteLine($"{syncInfo.Height} {syncInfo.TargetHeight}");

    foreach (var peer in await daemon.GetPeersAsync())
    {
        Console.WriteLine($"{peer.Host} {peer.Port} {peer.Height} {peer.IsIncoming}");
    }
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_sync_info> sync_info = daemon->get_sync_info();
    cout << sync_info->m_height.get() << " " << sync_info->m_target_height.get() << endl;

    for (const shared_ptr<monero_peer>& peer : daemon->get_peers()) {
      cout << peer->m_host.get() << " " << peer->m_port.get() << endl;
    }
    ```

=== "C"

    ```c
    char* sync_info = NULL;
    monero_daemon_get_sync_info(daemon, &sync_info);   // JSON, with the heights and the peers
    printf("%s\n", sync_info);
    monero_utils_free(sync_info);

    char* peers = NULL;
    monero_daemon_get_peers(daemon, &peers);           // JSON
    monero_utils_free(peers);
    ```
