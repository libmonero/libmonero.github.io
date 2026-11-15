# Peers and network

The peers of the node, who is banned, and the limits on its bandwidth and connections.

## See the peers

The peers that the node is connected to now, and the ones that it knows. A connected peer has a height, and tells if it connected to the node or the node to it. A known peer has the time when the node last saw it.

=== "Python"

    ```python
    for peer in daemon.get_peers():                # the connected peers
        print(peer.host, peer.port, peer.height, peer.is_incoming)

    for peer in daemon.get_known_peers():          # the peers in the list of the node
        print(peer.host, peer.port, peer.last_seen_timestamp)
    ```

=== "Java"

    ```java
    for (MoneroPeer peer : daemon.getPeers()) {    // the connected peers
      System.out.println(peer.getHost() + " " + peer.getPort() + " " + peer.getHeight() + " " + peer.isIncoming());
    }

    for (MoneroPeer peer : daemon.getKnownPeers()) {   // the peers in the list of the node
      System.out.println(peer.getHost() + " " + peer.getPort() + " " + peer.getLastSeenTimestamp());
    }
    ```

=== "Kotlin"

    ```kotlin
    for (peer in daemon.getPeers()) {              // the connected peers
        println("${peer.host} ${peer.port} ${peer.height} ${peer.isIncoming}")
    }

    for (peer in daemon.getKnownPeers()) {         // the peers in the list of the node
        println("${peer.host} ${peer.port} ${peer.lastSeenTimestamp}")
    }
    ```

=== "C#"

    ```csharp
    foreach (var peer in await daemon.GetPeersAsync())           // the connected peers
    {
        Console.WriteLine($"{peer.Host} {peer.Port} {peer.Height} {peer.IsIncoming}");
    }

    foreach (var peer in await daemon.GetKnownPeersAsync())      // the peers in the list of the node
    {
        Console.WriteLine($"{peer.Host} {peer.Port} {peer.LastSeenTimestamp}");
    }
    ```

=== "C++"

    ```c++
    for (const shared_ptr<monero_peer>& peer : daemon->get_peers()) {         // the connected peers
      cout << peer->m_host.get() << " " << peer->m_port.get() << " " << peer->m_height.get() << endl;
    }

    for (const shared_ptr<monero_peer>& peer : daemon->get_known_peers()) {   // the peers in the list of the node
      cout << peer->m_host.get() << " " << peer->m_port.get() << " " << peer->m_last_seen_timestamp.get() << endl;
    }
    ```

=== "C"

    ```c
    char* peers = NULL;
    monero_daemon_get_peers(daemon, &peers);               // JSON, the connected peers
    monero_utils_free(peers);

    char* known_peers = NULL;
    monero_daemon_get_known_peers(daemon, &known_peers);   // JSON, the peers in the list of the node
    monero_utils_free(known_peers);
    ```

The numbers of connections and the sync state of the node are in the [daemon status](status.md).

## Ban and unban a peer

A banned peer cannot connect to the node until the ban ends. Set `is_banned` to false to lift a ban before that. These calls change the node, and a node with a restricted RPC refuses them.

=== "Python"

    ```python
    ban = MoneroBan()
    ban.host = "192.168.1.100"
    ban.is_banned = True
    ban.seconds = 3600                       # for one hour
    daemon.set_peer_ban(ban)

    for ban in daemon.get_peer_bans():
        print(ban.host, ban.seconds)         # the seconds that are left

    ban.is_banned = False                    # lift it
    daemon.set_peer_ban(ban)
    ```

=== "Java"

    ```java
    MoneroBan ban = new MoneroBan()
            .setHost("192.168.1.100")
            .setIsBanned(true)
            .setSeconds(3600l);              // for one hour
    daemon.setPeerBan(ban);

    for (MoneroBan banned : daemon.getPeerBans()) {
      System.out.println(banned.getHost() + " " + banned.getSeconds());   // the seconds that are left
    }

    ban.setIsBanned(false);                  // lift it
    daemon.setPeerBan(ban);
    ```

=== "Kotlin"

    ```kotlin
    val ban = MoneroBan()
    ban.host = "192.168.1.100"
    ban.isBanned = true
    ban.seconds = 3600                       // for one hour
    daemon.setPeerBan(ban)

    for (banned in daemon.getPeerBans()) {
        println("${banned.host} ${banned.seconds}")   // the seconds that are left
    }

    ban.isBanned = false                     // lift it
    daemon.setPeerBan(ban)
    ```

=== "C#"

    ```csharp
    var ban = new MoneroBan
    {
        Host = "192.168.1.100",
        IsBanned = true,
        Seconds = 3600                       // for one hour
    };
    await daemon.SetPeerBanAsync(ban);

    foreach (var banned in await daemon.GetPeerBansAsync())
    {
        Console.WriteLine($"{banned.Host} {banned.Seconds}");   // the seconds that are left
    }

    ban.IsBanned = false;                    // lift it
    await daemon.SetPeerBanAsync(ban);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_ban> ban = make_shared<monero_ban>();
    ban->m_host = "192.168.1.100";
    ban->m_is_banned = true;
    ban->m_seconds = 3600;                   // for one hour
    daemon->set_peer_ban(ban);

    for (const shared_ptr<monero_ban>& banned : daemon->get_peer_bans()) {
      cout << banned->m_host.get() << " " << banned->m_seconds.get() << endl;   // the seconds that are left
    }

    ban->m_is_banned = false;                // lift it
    daemon->set_peer_ban(ban);
    ```

=== "C"

    ```c
    monero_daemon_set_peer_ban(daemon, "{\"host\":\"192.168.1.100\",\"isBanned\":true,\"seconds\":3600}");   // for one hour

    char* bans = NULL;
    monero_daemon_get_peer_bans(daemon, &bans);   // JSON, with the seconds that are left
    monero_utils_free(bans);

    monero_daemon_set_peer_ban(daemon, "{\"host\":\"192.168.1.100\",\"isBanned\":false}");   // lift it
    ```

## Limit the bandwidth and the peers

The limits of the node on what it downloads and uploads, in kB/s, and on how many peers connect. A setter returns the limit that is now in force, and a reset puts back the default of the node. These calls change the node, and a node with a restricted RPC refuses them.

=== "Python"

    ```python
    download_limit = daemon.get_download_limit()      # kB/s
    upload_limit = daemon.get_upload_limit()

    daemon.set_download_limit(2048)
    daemon.set_upload_limit(1024)
    daemon.reset_download_limit()                     # back to the default

    daemon.set_incoming_peer_limit(32)
    daemon.set_outgoing_peer_limit(16)
    ```

=== "Java"

    ```java
    int downloadLimit = daemon.getDownloadLimit();    // kB/s
    int uploadLimit = daemon.getUploadLimit();

    daemon.setDownloadLimit(2048);
    daemon.setUploadLimit(1024);
    daemon.resetDownloadLimit();                      // back to the default

    daemon.setIncomingPeerLimit(32);
    daemon.setOutgoingPeerLimit(16);
    ```

=== "Kotlin"

    ```kotlin
    val downloadLimit = daemon.getDownloadLimit()     // kB/s
    val uploadLimit = daemon.getUploadLimit()

    daemon.setDownloadLimit(2048)
    daemon.setUploadLimit(1024)
    daemon.resetDownloadLimit()                       // back to the default

    daemon.setIncomingPeerLimit(32)
    daemon.setOutgoingPeerLimit(16)
    ```

=== "C#"

    ```csharp
    int downloadLimit = await daemon.GetDownloadLimitAsync();    // kB/s
    int uploadLimit = await daemon.GetUploadLimitAsync();

    await daemon.SetDownloadLimitAsync(2048);
    await daemon.SetUploadLimitAsync(1024);
    await daemon.ResetDownloadLimitAsync();                      // back to the default

    await daemon.SetIncomingPeerLimitAsync(32);
    await daemon.SetOutgoingPeerLimitAsync(16);
    ```

=== "C++"

    ```c++
    int download_limit = daemon->get_download_limit();   // kB/s
    int upload_limit = daemon->get_upload_limit();

    daemon->set_download_limit(2048);
    daemon->set_upload_limit(1024);
    daemon->reset_download_limit();                      // back to the default

    daemon->set_incoming_peer_limit(32);
    daemon->set_outgoing_peer_limit(16);
    ```

=== "C"

    ```c
    int32_t limit = 0;
    monero_daemon_get_download_limit(daemon, &limit);    // kB/s
    monero_daemon_get_upload_limit(daemon, &limit);

    monero_daemon_set_download_limit(daemon, 2048, &limit);
    monero_daemon_set_upload_limit(daemon, 1024, &limit);
    monero_daemon_reset_download_limit(daemon, &limit);  // back to the default

    monero_daemon_set_incoming_peer_limit(daemon, 32);
    monero_daemon_set_outgoing_peer_limit(daemon, 16);
    ```
