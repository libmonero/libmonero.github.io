# Manage the node

Calls that change the node itself. A node with a restricted RPC refuses them, so they need a node of yours.

## Prune the blockchain

Pruning drops about two thirds of the chain from the disk of the node, and keeps what it needs to verify. It cannot be undone, and it takes a while. `check` set to true only tells if the blockchain is pruned, without pruning it.

=== "Python"

    ```python
    result = daemon.prune_blockchain(True)       # only check
    print(result.is_pruned, result.pruning_seed)

    result = daemon.prune_blockchain(False)      # prune it
    ```

=== "Java"

    ```java
    MoneroPruneResult result = daemon.pruneBlockchain(true);     // only check
    System.out.println(result.isPruned() + " " + result.getPruningSeed());

    result = daemon.pruneBlockchain(false);                       // prune it
    ```

=== "Kotlin"

    ```kotlin
    var result = daemon.pruneBlockchain(true)    // only check
    println("${result.isPruned} ${result.pruningSeed}")

    result = daemon.pruneBlockchain(false)       // prune it
    ```

=== "C#"

    ```csharp
    var result = await daemon.PruneBlockchainAsync(true);    // only check
    Console.WriteLine($"{result.IsPruned} {result.PruningSeed}");

    result = await daemon.PruneBlockchainAsync(false);       // prune it
    ```

=== "C++"

    ```c++
    shared_ptr<monero_prune_result> result = daemon->prune_blockchain(true);    // only check
    cout << result->m_is_pruned.get() << " " << result->m_pruning_seed.get() << endl;

    result = daemon->prune_blockchain(false);                                    // prune it
    ```

=== "C"

    ```c
    char* result = NULL;
    monero_daemon_prune_blockchain(daemon, true, &result);    // JSON, only check
    monero_utils_free(result);

    monero_daemon_prune_blockchain(daemon, false, &result);   // prune it
    monero_utils_free(result);
    ```

## Save the blockchain and set the log level

Save writes the database of the node to its disk. The log level goes from 0, the least, to 4, the most.

!!! note

    monero-java does not have these two calls.

=== "Python"

    ```python
    daemon.save_blockchain()

    daemon.set_log_level(1)
    ```

=== "Kotlin"

    ```kotlin
    daemon.saveBlockchain()

    daemon.setLogLevel(1)
    ```

=== "C#"

    ```csharp
    await daemon.SaveBlockchainAsync();

    await daemon.SetLogLevelAsync(1);
    ```

=== "C++"

    ```c++
    daemon->save_blockchain();

    daemon->set_log_level(1);
    ```

=== "C"

    ```c
    monero_daemon_save_blockchain(daemon);

    monero_daemon_set_log_level(daemon, 1);
    ```

## Stop the node

Ask the node to shut down. It does not start again by itself.

=== "Python"

    ```python
    daemon.stop()
    ```

=== "Java"

    ```java
    daemon.stop();
    ```

=== "Kotlin"

    ```kotlin
    daemon.stop()
    ```

=== "C#"

    ```csharp
    await daemon.StopAsync();
    ```

=== "C++"

    ```c++
    daemon->stop();
    ```

=== "C"

    ```c
    monero_daemon_stop(daemon);
    ```
