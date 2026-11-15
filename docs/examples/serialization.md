# Serialization

Every model of the libraries, a transaction, a block, a wallet config, a transaction config, can be turned into JSON and read back. The JSON has the same keys in every language, in camel case, such as `accountIndex` and `isConfirmed`. It is the one that monero-c uses at its boundary, so a model that one language writes is a model that another one reads.

Use it to keep a config in a file, to send a transaction from a service to another, to log what a call returned, or to hand a model to a program in another language.

!!! warning

    The JSON of a model has all that the model has. A wallet config can hold a seed and a password, and an RPC connection holds its login. Do not log them, and do not write them to a file that is not protected.

## Turn a model into JSON

=== "Python"

    ```python
    from monero import *

    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000

    json = tx_config.serialize()
    ```

=== "Java"

    ```java
    import common.utils.JsonUtils;

    MoneroTxConfig txConfig = new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000");

    String json = JsonUtils.serialize(txConfig);
    ```

=== "Kotlin"

    ```kotlin
    import monero.common.MoneroJson

    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))

    val json = MoneroJson.serialize(txConfig)
    ```

=== "C#"

    ```csharp
    using Monero.Common;

    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000
    };

    string json = MoneroJson.Serialize(txConfig);
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;

    string json = tx_config.serialize();
    ```

=== "C"

    ```c
    // the models of monero-c are JSON already: the config of a transaction is a string
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}]}", address);
    ```

## Read a model from JSON

Give the JSON and the type that it holds. Fields that the JSON does not have stay unset, and fields that the type does not know are ignored.

=== "Python"

    ```python
    tx_config = MoneroTxConfig.deserialize(json)

    print(tx_config.account_index, tx_config.address)
    ```

=== "Java"

    ```java
    MoneroTxConfig txConfig = JsonUtils.deserialize(json, MoneroTxConfig.class);

    System.out.println(txConfig.getAccountIndex() + " " + txConfig.getAddress());
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroJson.deserialize<MoneroTxConfig>(json)

    println("${txConfig.accountIndex} ${txConfig.destinations}")
    ```

=== "C#"

    ```csharp
    var txConfig = MoneroJson.Deserialize<MoneroTxConfig>(json);

    Console.WriteLine($"{txConfig.AccountIndex} {txConfig.Address}");
    ```

=== "C++"

    ```c++
    shared_ptr<monero_tx_config> tx_config = monero_tx_config::deserialize(json);

    cout << tx_config->m_account_index.get() << " " << tx_config->m_address.get() << endl;
    ```

=== "C"

    ```c
    // a call of monero-c takes the JSON as it is
    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);
    monero_utils_free(tx);
    ```

## Keep what a call returned

The result of a call, the state of a node for instance, is a model like the others. Serialize it to keep it or to send it. Every model that a call returns has its own `deserialize`, so the service that gets it can read it back as the same type.

=== "Python"

    ```python
    info = daemon.get_info()
    json = info.serialize()

    same_info = MoneroDaemonInfo.deserialize(json)
    ```

=== "Java"

    ```java
    MoneroDaemonInfo info = daemon.getInfo();
    String json = JsonUtils.serialize(info);

    MoneroDaemonInfo sameInfo = JsonUtils.deserialize(json, MoneroDaemonInfo.class);
    ```

=== "Kotlin"

    ```kotlin
    val info = daemon.getInfo()
    val json = MoneroJson.serialize(info)

    val sameInfo = MoneroJson.deserialize<MoneroDaemonInfo>(json)
    ```

=== "C#"

    ```csharp
    var info = await daemon.GetInfoAsync();
    string json = MoneroJson.Serialize(info);

    var sameInfo = MoneroJson.Deserialize<MoneroDaemonInfo>(json);
    ```

=== "C++"

    ```c++
    shared_ptr<monero_daemon_info> info = daemon->get_info();
    string json = info->serialize();

    shared_ptr<monero_daemon_info> same_info = monero_daemon_info::deserialize(json);
    ```

=== "C"

    ```c
    char* info = NULL;
    monero_daemon_get_info(daemon, &info);   // JSON, the state of the node
    // store it, or send it, as it is
    monero_utils_free(info);
    ```
