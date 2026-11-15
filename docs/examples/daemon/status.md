# Daemon status

What the node is doing: its release and network, how far it has synced, its connections, and whether it can be trusted.

## Get the state of the node

The release, the network, and where the node is on the chain, with the state of its connections and its pool. A node is ready when `is_synchronized` is true.

=== "Python"

    ```python
    info = daemon.get_info()

    print(info.version, info.network_type)
    print(info.height, info.target_height, info.is_synchronized)
    print(info.is_restricted, info.is_offline)
    print(info.num_incoming_connections, info.num_outgoing_connections)
    print(info.num_txs_pool, info.database_size)
    ```

=== "Java"

    ```java
    MoneroDaemonInfo info = daemon.getInfo();

    System.out.println(info.getVersion() + " " + info.getNetworkType());
    System.out.println(info.getHeight() + " " + info.getTargetHeight() + " " + info.isSynchronized());
    System.out.println(info.isRestricted() + " " + info.isOffline());
    System.out.println(info.getNumIncomingConnections() + " " + info.getNumOutgoingConnections());
    System.out.println(info.getNumTxsPool() + " " + info.getDatabaseSize());
    ```

=== "Kotlin"

    ```kotlin
    val info = daemon.getInfo()

    println("${info.version} ${info.networkType}")
    println("${info.height} ${info.targetHeight} ${info.isSynchronized}")
    println("${info.isRestricted} ${info.isOffline}")
    println("${info.numIncomingConnections} ${info.numOutgoingConnections}")
    println("${info.numTxsPool} ${info.databaseSize}")
    ```

=== "C#"

    ```csharp
    var info = await daemon.GetInfoAsync();

    Console.WriteLine($"{info.Version} {info.NetworkType}");
    Console.WriteLine($"{info.Height} {info.TargetHeight} {info.IsSynchronized}");
    Console.WriteLine($"{info.IsRestricted} {info.IsOffline}");
    Console.WriteLine($"{info.NumIncomingConnections} {info.NumOutgoingConnections}");
    Console.WriteLine($"{info.NumTxsPool} {info.DatabaseSize}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_info> info = daemon->get_info();

    cout << info->m_version.get() << endl;
    cout << info->m_height.get() << " " << info->m_target_height.get() << " " << info->m_is_synchronized.get() << endl;
    cout << info->m_is_restricted.get() << " " << info->m_is_offline.get() << endl;
    cout << info->m_num_incoming_connections.get() << " " << info->m_num_outgoing_connections.get() << endl;
    cout << info->m_num_txs_pool.get() << " " << info->m_database_size.get() << endl;
    ```

=== "C"

    ```c
    char* info = NULL;
    monero_daemon_get_info(daemon, &info);   // JSON, with the version, the network type, the heights, the connections and the pool
    printf("%s\n", info);
    monero_utils_free(info);
    ```

A node with `is_restricted` set serves the restricted RPC, as a public node does: it refuses the calls that change the node or that cost it a lot, such as [mining](mining.md) and [managing](manage.md) it.

## Get the sync state

How far the node is from the top of the chain, and how many peers it syncs from. While the node syncs, `height` grows toward `target_height`.

=== "Python"

    ```python
    sync_info = daemon.get_sync_info()

    print(sync_info.height, sync_info.target_height)
    print(len(sync_info.peers))
    ```

=== "Java"

    ```java
    MoneroDaemonSyncInfo syncInfo = daemon.getSyncInfo();

    System.out.println(syncInfo.getHeight() + " " + syncInfo.getTargetHeight());
    System.out.println(syncInfo.getPeers().size());
    ```

=== "Kotlin"

    ```kotlin
    val syncInfo = daemon.getSyncInfo()

    println("${syncInfo.height} ${syncInfo.targetHeight}")
    println(syncInfo.peers?.size)
    ```

=== "C#"

    ```csharp
    var syncInfo = await daemon.GetSyncInfoAsync();

    Console.WriteLine($"{syncInfo.Height} {syncInfo.TargetHeight}");
    Console.WriteLine(syncInfo.Peers?.Count);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_sync_info> sync_info = daemon->get_sync_info();

    cout << sync_info->m_height.get() << " " << sync_info->m_target_height.get() << endl;
    cout << sync_info->m_peers.size() << endl;
    ```

=== "C"

    ```c
    char* sync_info = NULL;
    monero_daemon_get_sync_info(daemon, &sync_info);   // JSON, with the heights and the peers
    printf("%s\n", sync_info);
    monero_utils_free(sync_info);
    ```

## Get the version of the RPC

The version of the RPC interface of the node, as a number, and whether it is a release. The release of the node itself is the `version` of its [state](#get-the-state-of-the-node).

=== "Python"

    ```python
    version = daemon.get_version()

    print(version.number, version.is_release)
    ```

=== "Java"

    ```java
    MoneroVersion version = daemon.getVersion();

    System.out.println(version.getNumber() + " " + version.isRelease());
    ```

=== "Kotlin"

    ```kotlin
    val version = daemon.getVersion()

    println("${version.number} ${version.isRelease}")
    ```

=== "C#"

    ```csharp
    var version = await daemon.GetVersionAsync();

    Console.WriteLine($"{version.Number} {version.IsRelease}");
    ```

=== "C++"

    ```c++
    monero_version version = daemon->get_version();

    cout << version.m_number.get() << " " << version.m_is_release.get() << endl;
    ```

=== "C"

    ```c
    char* version = NULL;
    monero_daemon_get_version(daemon, &version);   // JSON, with the number and isRelease
    printf("%s\n", version);
    monero_utils_free(version);
    ```

## Know if the node is trusted

Tell if the node is a trusted one. A client asks less of a node that it trusts, and more checks of one that it does not.

=== "Python"

    ```python
    if daemon.is_trusted():
        print("trusted")
    ```

=== "Java"

    ```java
    if (daemon.isTrusted()) {
      System.out.println("trusted");
    }
    ```

=== "Kotlin"

    ```kotlin
    if (daemon.isTrusted()) {
        println("trusted")
    }
    ```

=== "C#"

    ```csharp
    if (await daemon.IsTrustedAsync())
    {
        Console.WriteLine("trusted");
    }
    ```

=== "C++"

    ```c++
    if (daemon->is_trusted()) {
      cout << "trusted" << endl;
    }
    ```

=== "C"

    ```c
    bool trusted = false;
    monero_daemon_is_trusted(daemon, &trusted);
    if (trusted) printf("trusted\n");
    ```

## Check for an update

Ask the node if a newer release exists. It only checks: it does not download or install anything.

=== "Python"

    ```python
    update = daemon.check_for_update()

    if update.is_update_available:
        print(update.version, update.user_uri)
    ```

=== "Java"

    ```java
    MoneroDaemonUpdateCheckResult update = daemon.checkForUpdate();

    if (update.isUpdateAvailable()) {
      System.out.println(update.getVersion() + " " + update.getUserUri());
    }
    ```

=== "Kotlin"

    ```kotlin
    val update = daemon.checkForUpdate()

    if (update.isUpdateAvailable == true) {
        println("${update.version} ${update.userUri}")
    }
    ```

=== "C#"

    ```csharp
    var update = await daemon.CheckForUpdateAsync();

    if (update.IsUpdateAvailable == true)
    {
        Console.WriteLine($"{update.Version} {update.UserUri}");
    }
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_update_check_result> update = daemon->check_for_update();

    if (update->m_is_update_available.get()) {
      cout << update->m_version.get() << " " << update->m_user_uri.get() << endl;
    }
    ```

=== "C"

    ```c
    char* update = NULL;
    monero_daemon_check_for_update(daemon, &update);   // JSON, with isUpdateAvailable, the version and the URIs
    printf("%s\n", update);
    monero_utils_free(update);
    ```
