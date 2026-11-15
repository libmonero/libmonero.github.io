# Blockchain

Read the chain: the height, the blocks and their headers, the forks and the hard fork, and the new blocks as they come. The examples expect the node of the [overview](../index.md#run-a-node-and-a-wallet-server) on `localhost:38081`, and a `daemon` that is connected to it.

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

## Get a block by its hash

A block is found by its height or by its hash. The hash of the block at a height is a call of its own, and it is lighter than the block.

=== "Python"

    ```python
    block_hash = daemon.get_block_hash(height - 1)    # the hash of the block at a height

    block = daemon.get_block_by_hash(block_hash)
    print(block.height, block.hash, block.tx_hashes)
    ```

=== "Java"

    ```java
    String blockHash = daemon.getBlockHash(height - 1);    // the hash of the block at a height

    MoneroBlock block = daemon.getBlockByHash(blockHash);
    System.out.println(block.getHeight() + " " + block.getHash() + " " + block.getTxHashes());
    ```

=== "Kotlin"

    ```kotlin
    val blockHash = daemon.getBlockHash(height - 1)    // the hash of the block at a height

    val block = daemon.getBlockByHash(blockHash)
    println("${block.height} ${block.hash} ${block.txHashes}")
    ```

=== "C#"

    ```csharp
    string blockHash = await daemon.GetBlockHashAsync(height - 1);    // the hash of the block at a height

    var block = await daemon.GetBlockByHashAsync(blockHash);
    Console.WriteLine($"{block.Height} {block.Hash} {block.TxHashes?.Count}");
    ```

=== "C++"

    ```c++
    string block_hash = daemon->get_block_hash(height - 1);    // the hash of the block at a height

    shared_ptr<monero_block> block = daemon->get_block_by_hash(block_hash);
    cout << block->m_height.get() << " " << block->m_hash.get() << endl;
    monero_utils::free(block);
    ```

=== "C"

    ```c
    char* block_hash = NULL;
    monero_daemon_get_block_hash(daemon, height - 1, &block_hash);    // the hash of the block at a height

    char* block = NULL;
    monero_daemon_get_block_by_hash(daemon, block_hash, &block);      // JSON
    printf("%s\n", block);

    monero_utils_free(block);
    monero_utils_free(block_hash);
    ```

## Get a range of blocks

The blocks of a range come with their transactions, so keep the range short. To scan a long stretch of the chain, read the [headers](#get-block-headers), which are light, and get the blocks that you need.

=== "Python"

    ```python
    blocks = daemon.get_blocks_by_range(height - 4, height - 1)    # 4 blocks

    for block in blocks:
        print(block.height, len(block.txs))
    ```

=== "Java"

    ```java
    List<MoneroBlock> blocks = daemon.getBlocksByRange(height - 4, height - 1);    // 4 blocks

    for (MoneroBlock block : blocks) {
      System.out.println(block.getHeight() + " " + block.getTxs().size());
    }
    ```

=== "Kotlin"

    ```kotlin
    val blocks = daemon.getBlocksByRange(height - 4, height - 1)    // 4 blocks

    for (block in blocks) {
        println("${block.height} ${block.txs?.size}")
    }
    ```

=== "C#"

    ```csharp
    var blocks = await daemon.GetBlocksByRangeAsync(height - 4, height - 1);    // 4 blocks

    foreach (var block in blocks)
    {
        Console.WriteLine($"{block.Height} {block.Txs?.Count}");
    }
    ```

=== "C++"

    ```c++
    vector<shared_ptr<monero_block>> blocks = daemon->get_blocks_by_range(height - 4, height - 1);    // 4 blocks

    for (const shared_ptr<monero_block>& block : blocks) {
      cout << block->m_height.get() << " " << block->m_txs.size() << endl;
      monero_utils::free(block);
    }
    ```

=== "C"

    ```c
    uint64_t start_height = height - 4;
    uint64_t end_height = height - 1;

    char* blocks = NULL;
    monero_daemon_get_blocks_by_range(daemon, &start_height, &end_height, &blocks);    // JSON, 4 blocks
    printf("%s\n", blocks);
    monero_utils_free(blocks);
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

## Look at alternative chains

A node keeps the blocks of the forks that it has seen and that are not part of the main chain. Most of the time there are none.

=== "Python"

    ```python
    for chain in daemon.get_alt_chains():
        print(chain.height, chain.length, chain.main_chain_parent_block_hash)

    alt_block_hashes = daemon.get_alt_block_hashes()     # the hashes of all the blocks of the forks
    ```

=== "Java"

    ```java
    for (MoneroAltChain chain : daemon.getAltChains()) {
      System.out.println(chain.getHeight() + " " + chain.getLength() + " " + chain.getMainChainParentBlockHash());
    }

    List<String> altBlockHashes = daemon.getAltBlockHashes();   // the hashes of all the blocks of the forks
    ```

=== "Kotlin"

    ```kotlin
    for (chain in daemon.getAltChains()) {
        println("${chain.height} ${chain.length} ${chain.mainChainParentBlockHash}")
    }

    val altBlockHashes = daemon.getAltBlockHashes()   // the hashes of all the blocks of the forks
    ```

=== "C#"

    ```csharp
    foreach (var chain in await daemon.GetAltChainsAsync())
    {
        Console.WriteLine($"{chain.Height} {chain.Length} {chain.MainChainParentBlockHash}");
    }

    var altBlockHashes = await daemon.GetAltBlockHashesAsync();   // the hashes of all the blocks of the forks
    ```

=== "C++"

    ```c++
    for (const shared_ptr<monero_alt_chain>& chain : daemon->get_alt_chains()) {
      cout << chain->m_height.get() << " " << chain->m_length.get() << endl;
    }

    vector<string> alt_block_hashes = daemon->get_alt_block_hashes();   // the hashes of all the blocks of the forks
    ```

=== "C"

    ```c
    char* chains = NULL;
    monero_daemon_get_alt_chains(daemon, &chains);               // JSON, an empty list when there are no forks
    monero_utils_free(chains);

    char* alt_block_hashes = NULL;
    monero_daemon_get_alt_block_hashes(daemon, &alt_block_hashes);   // JSON array
    monero_utils_free(alt_block_hashes);
    ```

## Check the hard fork

The state of the hard fork that the node is on: the version of the rules, whether they are enabled, and where they start.

=== "Python"

    ```python
    fork = daemon.get_hard_fork_info()

    print(fork.version, fork.is_enabled)
    print(fork.earliest_height, fork.window, fork.threshold)
    ```

=== "Java"

    ```java
    MoneroHardForkInfo fork = daemon.getHardForkInfo();

    System.out.println(fork.getVersion() + " " + fork.isEnabled());
    System.out.println(fork.getEarliestHeight() + " " + fork.getWindow() + " " + fork.getThreshold());
    ```

=== "Kotlin"

    ```kotlin
    val fork = daemon.getHardForkInfo()

    println("${fork.version} ${fork.isEnabled}")
    println("${fork.earliestHeight} ${fork.window} ${fork.threshold}")
    ```

=== "C#"

    ```csharp
    var fork = await daemon.GetHardForkInfoAsync();

    Console.WriteLine($"{fork.Version} {fork.IsEnabled}");
    Console.WriteLine($"{fork.EarliestHeight} {fork.Window} {fork.Threshold}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_hard_fork_info> fork = daemon->get_hard_fork_info();

    cout << fork->m_version.get() << " " << fork->m_is_enabled.get() << endl;
    cout << fork->m_earliest_height.get() << " " << fork->m_window.get() << " " << fork->m_threshold.get() << endl;
    ```

=== "C"

    ```c
    char* fork = NULL;
    monero_daemon_get_hard_fork_info(daemon, &fork);   // JSON, with the version, isEnabled, earliestHeight, window and threshold
    printf("%s\n", fork);
    monero_utils_free(fork);
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


The same kind of notification exists for a wallet, see [Notifications](../notifications.md).
