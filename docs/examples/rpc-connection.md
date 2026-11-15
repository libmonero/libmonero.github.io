# RPC connection

The daemon and wallet RPC clients talk to their server through a connection: a URI, a login, and optionally a proxy and a timeout. The other pages pass the URI and the login to the client. This page shows the connection itself, which you build once, check, and share between clients.

The examples expect the servers of the [overview](index.md#run-a-node-and-a-wallet-server).

## Create a connection and give it to a client

A client made from a URI makes its own connection. Make the connection yourself when you want to set more than the login, or to use it for more than one client.

=== "Python"

    ```python
    from monero import *

    daemon_connection = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    daemon = MoneroDaemonRpc(daemon_connection)

    wallet_connection = MoneroRpcConnection("http://localhost:38083", "rpc_user", "abc123")
    wallet = MoneroWalletRpc(wallet_connection)
    ```

=== "Java"

    ```java
    import monero.common.MoneroRpcConnection;
    import monero.daemon.MoneroDaemonRpc;
    import monero.wallet.MoneroWalletRpc;

    MoneroRpcConnection daemonConnection = new MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123");
    MoneroDaemonRpc daemon = new MoneroDaemonRpc(daemonConnection);

    MoneroRpcConnection walletConnection = new MoneroRpcConnection("http://localhost:38083", "rpc_user", "abc123");
    MoneroWalletRpc wallet = new MoneroWalletRpc(walletConnection);
    ```

=== "Kotlin"

    ```kotlin
    import monero.common.MoneroRpcConnection
    import monero.daemon.MoneroDaemonRpc
    import monero.wallet.MoneroWalletRpc

    val daemonConnection = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    val daemon = MoneroDaemonRpc(daemonConnection)

    val walletConnection = MoneroRpcConnection("http://localhost:38083", "rpc_user", "abc123")
    val wallet = MoneroWalletRpc(walletConnection)
    ```

=== "C#"

    ```csharp
    using Monero.Common;
    using Monero.Daemon;
    using Monero.Wallet;

    var daemonConnection = new MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123");
    var daemon = new MoneroDaemonRpc(daemonConnection);

    var walletConnection = new MoneroRpcConnection("http://localhost:38083", "rpc_user", "abc123");
    await using var wallet = new MoneroWalletRpc(walletConnection);
    ```

=== "C++"

    ```c++
    using namespace monero;

    shared_ptr<monero_rpc_connection> daemon_connection = make_shared<monero_rpc_connection>("http://localhost:38081", "superuser", "abctesting123");
    monero_daemon_rpc* daemon = new monero_daemon_rpc(daemon_connection);

    shared_ptr<monero_rpc_connection> wallet_connection = make_shared<monero_rpc_connection>("http://localhost:38083", "rpc_user", "abc123");
    monero_wallet_rpc* wallet = new monero_wallet_rpc(wallet_connection);

    delete wallet;
    delete daemon;
    ```

=== "C"

    ```c
    #include "monero_c.h"

    // a connection is built from its JSON
    monero_rpc_connection* daemon_connection = NULL;
    monero_rpc_connection_create("{\"uri\":\"http://localhost:38081\",\"username\":\"superuser\",\"password\":\"abctesting123\"}", &daemon_connection);
    monero_daemon* daemon = NULL;
    monero_daemon_connect_with(daemon_connection, &daemon);

    monero_rpc_connection* wallet_connection = NULL;
    monero_rpc_connection_create("{\"uri\":\"http://localhost:38083\",\"username\":\"rpc_user\",\"password\":\"abc123\"}", &wallet_connection);
    monero_wallet* wallet = NULL;
    monero_wallet_rpc_connect(wallet_connection, &wallet);

    monero_wallet_free(wallet);
    monero_daemon_free(daemon);
    monero_rpc_connection_free(wallet_connection);
    monero_rpc_connection_free(daemon_connection);
    ```

## Check that the server is up

A connection does not call the server when it is made. Ask it to check, and it tells if the server answers and accepts the login. Until the first check, the answers are unknown: `None`, `null` or unset, depending on the language.

=== "Python"

    ```python
    connection.check_connection()     # calls the server, True if the state changed since the last check

    connection.is_online()            # True when the server answers
    connection.is_authenticated()     # True when the server accepts the login
    connection.is_connected()         # True when it is online and authenticated
    ```

=== "Java"

    ```java
    connection.checkConnection(5000l);  // calls the server, true if the state changed, waits up to 5 seconds

    connection.isOnline();              // true when the server answers
    connection.isAuthenticated();       // true when the server accepts the login
    connection.isConnected();           // true when it is online and authenticated
    ```

=== "Kotlin"

    ```kotlin
    connection.checkConnection(5000)    // calls the server, true if the state changed, waits up to 5 seconds

    connection.isOnline                 // true when the server answers
    connection.isAuthenticated          // true when the server accepts the login
    connection.isConnected()            // true when it is online and authenticated
    ```

=== "C#"

    ```csharp
    // the connection of monero-csharp does not call the server by itself:
    // a call of the client tells if the server is up and accepts the login
    try
    {
        ulong height = await daemon.GetHeightAsync();
        Console.WriteLine($"online at height {height}");
    }
    catch (MoneroError e)
    {
        Console.WriteLine($"not connected: {e.Message}");
    }
    ```

=== "C++"

    ```c++
    connection->check_connection(5000);   // calls the server, true if the state changed, waits up to 5 seconds

    connection->is_online();              // optional<bool>: true when the server answers
    connection->is_authenticated();       // true when the server accepts the login
    connection->is_connected();           // true when it is online and authenticated
    ```

=== "C"

    ```c
    uint32_t timeout_ms = 5000;
    bool changed = false;
    monero_rpc_connection_check_connection(connection, &timeout_ms, &changed);  // calls the server

    monero_optional_bool connected = MONERO_OPTIONAL_BOOL_UNSET;
    monero_rpc_connection_is_connected(connection, &connected);                 // TRUE when it is online and authenticated
    ```

## Connect a wallet to a daemon

A full wallet needs a daemon to sync with, and a wallet on a wallet RPC server uses the one that the server was started with, until you set another. A connection to a daemon can change while the wallet is open.

=== "Python"

    ```python
    wallet.set_daemon_connection("http://localhost:38081", "superuser", "abctesting123")

    wallet.is_connected_to_daemon()       # True when the daemon answers
    connection = wallet.get_daemon_connection()
    print(connection.uri)
    ```

=== "Java"

    ```java
    wallet.setDaemonConnection("http://localhost:38081", "superuser", "abctesting123");

    wallet.isConnectedToDaemon();         // true when the daemon answers
    MoneroRpcConnection connection = wallet.getDaemonConnection();
    System.out.println(connection.getUri());
    ```

=== "Kotlin"

    ```kotlin
    wallet.setDaemonConnection("http://localhost:38081", "superuser", "abctesting123")

    wallet.isConnectedToDaemon()          // true when the daemon answers
    val connection = wallet.getDaemonConnection()
    println(connection?.uri)
    ```

=== "C#"

    ```csharp
    await wallet.SetDaemonConnectionAsync("http://localhost:38081", "superuser", "abctesting123");

    bool connected = await wallet.IsConnectedToDaemonAsync();   // true when the daemon answers
    MoneroRpcConnection? connection = await wallet.GetDaemonConnectionAsync();
    Console.WriteLine(connection?.Uri);
    ```

=== "C++"

    ```c++
    wallet->set_daemon_connection("http://localhost:38081", "superuser", "abctesting123");

    bool connected = wallet->is_connected_to_daemon();   // true when the daemon answers
    shared_ptr<monero_rpc_connection> connection = wallet->get_daemon_connection();
    cout << connection->m_uri.get() << endl;
    ```

=== "C"

    ```c
    // NULL trusts the daemon only if it runs on this machine
    monero_wallet_set_daemon_connection(wallet, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);

    bool connected = false;
    monero_wallet_is_connected_to_daemon(wallet, &connected);   // true when the daemon answers

    char* connection = NULL;
    monero_wallet_get_daemon_connection(wallet, &connection);   // JSON of the connection
    monero_utils_free(connection);
    ```

## Connect through Tor

A connection can go through a SOCKS proxy, such as the one of a Tor node, which has to be running already and listens on `127.0.0.1:9050` by default. The proxy has the form `[scheme://]ip:port`, with the host as an IP address and not a name such as `localhost`. The default scheme and `socks5` let the proxy resolve an `.onion` address, and `socks4` does not. Tor is slow, so give the connection more time.

=== "Python"

    ```python
    connection = MoneroRpcConnection(
        "http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081",
        "superuser", "abctesting123",
        proxy_uri="127.0.0.1:9050",
        timeout_ms=60000)

    connection.is_onion()                 # True
    daemon = MoneroDaemonRpc(connection)
    ```

=== "Java"

    ```java
    MoneroRpcConnection connection = new MoneroRpcConnection("http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081")
            .setCredentials("superuser", "abctesting123")
            .setProxyUri("127.0.0.1:9050");

    connection.isOnion();                 // true
    MoneroDaemonRpc daemon = new MoneroDaemonRpc(connection);
    ```

=== "Kotlin"

    ```kotlin
    val connection = MoneroRpcConnection(
        "http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081",
        "superuser", "abctesting123")
    connection.proxyUri = "127.0.0.1:9050"
    connection.timeoutMs = 60000

    connection.isOnion()                  // true
    val daemon = MoneroDaemonRpc(connection)
    ```

=== "C#"

    ```csharp
    var connection = new MoneroRpcConnection(
        "http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081",
        "superuser", "abctesting123")
    {
        ProxyUri = "127.0.0.1:9050",
        TimeoutMs = 60000
    };

    var daemon = new MoneroDaemonRpc(connection);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_rpc_connection> connection = make_shared<monero_rpc_connection>(
        "http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081",
        "superuser", "abctesting123", "127.0.0.1:9050");
    connection->m_timeout_ms = 60000;

    monero_daemon_rpc* daemon = new monero_daemon_rpc(connection);
    ```

=== "C"

    ```c
    monero_daemon* daemon = NULL;
    monero_daemon_connect("http://kyaklhqp4yyza4fmtbvs6z4lrzr5cljxwa7o2d42sfelfhczsmbwzfad.onion:18081", "superuser", "abctesting123",
                          "127.0.0.1:9050", "", 60000, &daemon);
    ```

The proxy is not checked when it is set: a mistake in it shows as a network error at the first call.
