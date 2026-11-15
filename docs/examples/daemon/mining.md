# Mining

The mining of the node itself, and what a miner asks of it. A node with a restricted RPC, as a public node has, refuses most of these calls.

## Get the mining status

Whether the node is mining, with which address and how many threads, and how fast.

=== "Python"

    ```python
    status = daemon.get_mining_status()

    print(status.is_active, status.address)
    print(status.num_threads, status.speed, status.is_background)
    ```

=== "Java"

    ```java
    MoneroMiningStatus status = daemon.getMiningStatus();

    System.out.println(status.isActive() + " " + status.getAddress());
    System.out.println(status.getNumThreads() + " " + status.getSpeed() + " " + status.isBackground());
    ```

=== "Kotlin"

    ```kotlin
    val status = daemon.getMiningStatus()

    println("${status.isActive} ${status.address}")
    println("${status.numThreads} ${status.speed} ${status.isBackground}")
    ```

=== "C#"

    ```csharp
    var status = await daemon.GetMiningStatusAsync();

    Console.WriteLine($"{status.IsActive} {status.Address}");
    Console.WriteLine($"{status.NumThreads} {status.Speed} {status.IsBackground}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_mining_status> status = daemon->get_mining_status();

    cout << status->m_is_active.get() << " " << status->m_address.get() << endl;
    cout << status->m_num_threads.get() << " " << status->m_speed.get() << endl;
    ```

=== "C"

    ```c
    char* status = NULL;
    monero_daemon_get_mining_status(daemon, &status);   // JSON, with isActive, the address, the threads and the speed
    printf("%s\n", status);
    monero_utils_free(status);
    ```

## Start and stop mining

The node mines to an address, with the number of threads that you give it. A node with a restricted RPC refuses this, and so does a public one. The reward goes to the address, so use one of a test wallet.

=== "Python"

    ```python
    daemon.start_mining(address, 1, False, False)   # 1 thread, not in the background, do not ignore the battery

    daemon.stop_mining()
    ```

=== "Java"

    ```java
    daemon.startMining(address, 1l, false, false);   // 1 thread, not in the background, do not ignore the battery

    daemon.stopMining();
    ```

=== "Kotlin"

    ```kotlin
    daemon.startMining(address, 1)                   // 1 thread

    daemon.stopMining()
    ```

=== "C#"

    ```csharp
    await daemon.StartMiningAsync(address, 1);       // 1 thread

    await daemon.StopMiningAsync();
    ```

=== "C++"

    ```c++
    daemon->start_mining(address, 1, false, false);  // 1 thread, not in the background, do not ignore the battery

    daemon->stop_mining();
    ```

=== "C"

    ```c
    uint64_t num_threads = 1;
    bool is_background = false;
    bool ignore_battery = false;
    monero_daemon_start_mining(daemon, address, &num_threads, &is_background, &ignore_battery);

    monero_daemon_stop_mining(daemon);
    ```

## Get a block template

The block that a miner would work on now: where it builds on, its height and its expected reward. A mining pool, or a miner that mines outside the node, asks for it.

=== "Python"

    ```python
    template = daemon.get_block_template(address)

    print(template.height, template.prev_hash)
    print(template.expected_reward)
    ```

=== "Java"

    ```java
    MoneroBlockTemplate template = daemon.getBlockTemplate(address);

    System.out.println(template.getHeight() + " " + template.getPrevHash());
    System.out.println(template.getExpectedReward());
    ```

=== "Kotlin"

    ```kotlin
    val template = daemon.getBlockTemplate(address)

    println("${template.height} ${template.prevHash}")
    println(template.expectedReward)
    ```

=== "C#"

    ```csharp
    var template = await daemon.GetBlockTemplateAsync(address);

    Console.WriteLine($"{template.Height} {template.PrevHash}");
    Console.WriteLine(template.ExpectedReward);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_block_template> block_template = daemon->get_block_template(address);

    cout << block_template->m_height.get() << " " << block_template->m_prev_hash.get() << endl;
    cout << block_template->m_expected_reward.get() << endl;
    ```

=== "C"

    ```c
    char* block_template = NULL;
    monero_daemon_get_block_template(daemon, address, NULL, &block_template);   // JSON, with the height, the previous hash and the reward
    printf("%s\n", block_template);
    monero_utils_free(block_template);
    ```

## Generate blocks on regtest

A regtest node mines a block when you ask for one, with no waiting: it is how the tests of the libraries get funds. The reward goes to the address, and a coinbase output is locked for 60 blocks, so generate more than that before you spend it.

!!! note

    monero-java does not have this call. Start the node with `--regtest`: the [regtest compose file](https://github.com/libmonero/docker-monero-rpc/blob/main/examples/regtest/docker-compose.yml) of docker-monero-rpc does.

=== "Python"

    ```python
    result = daemon.generate_blocks(address, 70)

    print(result.height, len(result.block_hashes))
    ```

=== "Kotlin"

    ```kotlin
    val result = daemon.generateBlocks(address, 70)

    println("${result.height} ${result.blockHashes?.size}")
    ```

=== "C#"

    ```csharp
    var result = await daemon.GenerateBlocksAsync(address, 70);

    Console.WriteLine($"{result.Height} {result.BlockHashes?.Count}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_generate_blocks_result> result = daemon->generate_blocks(address, 70);

    cout << result->m_height.get() << " " << result->m_block_hashes.size() << endl;
    ```

=== "C"

    ```c
    char* result = NULL;
    monero_daemon_generate_blocks(daemon, address, 70, NULL, NULL, &result);   // JSON, with the height and the hashes of the blocks
    printf("%s\n", result);
    monero_utils_free(result);
    ```
