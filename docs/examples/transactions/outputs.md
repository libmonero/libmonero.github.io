# Outputs and sweeping

The outputs of a wallet are the pieces of funds that it can spend. List them, move all of them or only one, clean up the dust, and keep one out of the way. The examples expect a `wallet` with funds.

## List the outputs

The outputs of a wallet are the pieces of funds that it can spend. Ask for the unspent ones of an account, with their amount and the subaddress that received them. The key image identifies an output, and is what the calls below take.

=== "Python"

    ```python
    query = MoneroOutputQuery()
    query.is_spent = False
    query.account_index = 0

    for output in wallet.get_outputs(query):
        print(output.amount, output.subaddress_index, output.is_frozen, output.key_image.hex)
    ```

=== "Java"

    ```java
    MoneroOutputQuery query = new MoneroOutputQuery();
    query.setIsSpent(false);
    query.setAccountIndex(0);

    for (MoneroOutputWallet output : wallet.getOutputs(query)) {
      System.out.println(output.getAmount() + " " + output.getSubaddressIndex() + " " + output.isFrozen() + " " + output.getKeyImage().getHex());
    }
    ```

=== "Kotlin"

    ```kotlin
    val query = MoneroOutputQuery()
    query.isSpent = false
    query.accountIndex = 0

    for (output in wallet.getOutputs(query)) {
        println("${output.amount} ${output.subaddressIndex} ${output.isFrozen} ${output.keyImage?.hex}")
    }
    ```

=== "C#"

    ```csharp
    var query = new MoneroOutputQuery { IsSpent = false, AccountIndex = 0 };

    foreach (var output in await wallet.GetOutputsAsync(query))
    {
        Console.WriteLine($"{output.Amount} {output.SubaddressIndex} {output.IsFrozen} {output.KeyImage?.Hex}");
    }
    ```

=== "C++"

    ```c++
    monero_output_query query;
    query.m_is_spent = false;
    query.m_account_index = 0;

    vector<shared_ptr<monero_output_wallet>> outputs = wallet->get_outputs(query);
    for (const shared_ptr<monero_output_wallet>& output : outputs) {
      cout << output->m_amount.get() << " " << output->m_subaddress_index.get() << " " << output->m_key_image->m_hex.get() << endl;
    }
    monero_utils::free(outputs);
    ```

=== "C"

    ```c
    char* outputs = NULL;
    monero_wallet_get_outputs(wallet, "{\"isSpent\":false,\"accountIndex\":0}", &outputs);   // JSON, the outputs in their blocks
    printf("%s\n", outputs);
    monero_utils_free(outputs);
    ```

## Sweep all the funds to an address

A sweep sends everything that is unlocked, with no amount to compute and no change. Use it to empty an account, or to move the funds of a wallet to a new one. A sweep can need several transactions, so it returns a list.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.relay = False

    txs = wallet.sweep_unlocked(tx_config)
    for tx in txs:
        print(tx.hash, tx.fee)

    wallet.relay_txs(txs)
    ```

=== "Java"

    ```java
    List<MoneroTxWallet> txs = wallet.sweepUnlocked(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setRelay(false));

    for (MoneroTxWallet tx : txs) {
      System.out.println(tx.getHash() + " " + tx.getFee());
    }

    wallet.relayTxs(txs);
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.address = address
    txConfig.relay = false

    val txs = wallet.sweepUnlocked(txConfig)
    for (tx in txs) {
        println("${tx.hash} ${tx.fee}")
    }

    wallet.relayTxs(txs)
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Relay = false
    };

    var txs = await wallet.SweepUnlockedAsync(txConfig);
    foreach (var tx in txs)
    {
        Console.WriteLine($"{tx.Hash} {tx.Fee}");
    }

    await wallet.RelayTxsAsync(txs);
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_relay = false;

    vector<shared_ptr<monero_tx_wallet>> txs = wallet->sweep_unlocked(tx_config);
    for (const shared_ptr<monero_tx_wallet>& tx : txs) {
      cout << tx->m_hash.get() << " " << tx->m_fee.get() << endl;
    }

    wallet->relay_txs(txs);
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config), "{\"accountIndex\":0,\"address\":\"%s\",\"relay\":false}", address);

    char* txs = NULL;
    monero_wallet_sweep_unlocked(wallet, tx_config, &txs);   // JSON, the transaction sets

    printf("%s\n", txs);
    monero_utils_free(txs);
    ```

To sweep only some of the funds, give the subaddresses with `subaddress_indices`. Set `sweep_each_subaddress` to sweep each of them to the address in a transaction of its own.

## Sweep one output

Send a single output, chosen by its key image, with the funds of no other. It is how you move one output that you picked from the [list](#list-the-outputs).

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.address = address
    tx_config.key_image = key_image            # the hex of the key image of the output
    tx_config.relay = False

    tx = wallet.sweep_output(tx_config)
    print(tx.hash, tx.fee)

    wallet.relay_tx(tx)
    ```

=== "Java"

    ```java
    MoneroTxWallet tx = wallet.sweepOutput(new MoneroTxConfig()
            .setAddress(address)
            .setKeyImage(keyImage)             // the hex of the key image of the output
            .setRelay(false));

    System.out.println(tx.getHash() + " " + tx.getFee());

    wallet.relayTx(tx);
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.address = address
    txConfig.keyImage = keyImage               // the hex of the key image of the output
    txConfig.relay = false

    val tx = wallet.sweepOutput(txConfig)
    println("${tx.hash} ${tx.fee}")

    wallet.relayTx(tx)
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        Address = address,
        KeyImage = keyImage,                   // the hex of the key image of the output
        Relay = false
    };

    MoneroTxWallet tx = await wallet.SweepOutputAsync(txConfig);
    Console.WriteLine($"{tx.Hash} {tx.Fee}");

    await wallet.RelayTxAsync(tx);
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_address = address;
    tx_config.m_key_image = key_image;         // the hex of the key image of the output
    tx_config.m_relay = false;

    shared_ptr<monero_tx_wallet> tx = wallet->sweep_output(tx_config);
    cout << tx->m_hash.get() << " " << tx->m_fee.get() << endl;

    wallet->relay_tx(*tx);
    monero_utils::free(tx);
    ```

=== "C"

    ```c
    char tx_config[512];
    snprintf(tx_config, sizeof(tx_config), "{\"address\":\"%s\",\"keyImage\":\"%s\",\"relay\":false}", address, key_image);

    char* tx = NULL;
    monero_wallet_sweep_output(wallet, tx_config, &tx);   // JSON

    char* hash = NULL;
    monero_wallet_relay_tx_json(wallet, tx, &hash);

    monero_utils_free(hash);
    monero_utils_free(tx);
    ```

## Sweep the dust

Dust is the outputs that are too small to be worth spending. Sweeping them sends them back to the wallet itself, in fewer outputs. The result is a list of transactions, and nothing is sent until you relay them, unless you ask for it.

=== "Python"

    ```python
    txs = wallet.sweep_dust(False)       # False: create them, do not relay them yet
    print(len(txs))

    wallet.relay_txs(txs)
    ```

=== "Java"

    ```java
    List<MoneroTxWallet> txs = wallet.sweepDust(false);   // false: create them, do not relay them yet
    System.out.println(txs.size());

    wallet.relayTxs(txs);
    ```

=== "Kotlin"

    ```kotlin
    val txs = wallet.sweepDust(false)    // false: create them, do not relay them yet
    println(txs.size)

    wallet.relayTxs(txs)
    ```

=== "C#"

    ```csharp
    var txs = await wallet.SweepDustAsync(false);    // false: create them, do not relay them yet
    Console.WriteLine(txs.Count);

    await wallet.RelayTxsAsync(txs);
    ```

=== "C++"

    ```c++
    vector<shared_ptr<monero_tx_wallet>> txs = wallet->sweep_dust(false);   // false: create them, do not relay them yet
    cout << txs.size() << endl;

    wallet->relay_txs(txs);
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char* txs = NULL;
    monero_wallet_sweep_dust(wallet, false, &txs);   // JSON, false: create them, do not relay them yet
    printf("%s\n", txs);
    monero_utils_free(txs);
    ```

## Freeze and thaw an output

A frozen output is left out of every transaction that the wallet creates, so it stays where it is. Freeze an output that you do not want to spend by mistake, such as one that links to an identity. Thaw it to spend it again.

=== "Python"

    ```python
    wallet.freeze_output(key_image)

    print(wallet.is_output_frozen(key_image))    # True

    wallet.thaw_output(key_image)
    ```

=== "Java"

    ```java
    wallet.freezeOutput(keyImage);

    System.out.println(wallet.isOutputFrozen(keyImage));    // true

    wallet.thawOutput(keyImage);
    ```

=== "Kotlin"

    ```kotlin
    wallet.freezeOutput(keyImage)

    println(wallet.isOutputFrozen(keyImage))     // true

    wallet.thawOutput(keyImage)
    ```

=== "C#"

    ```csharp
    await wallet.FreezeOutputAsync(keyImage);

    Console.WriteLine(await wallet.IsOutputFrozenAsync(keyImage));    // True

    await wallet.ThawOutputAsync(keyImage);
    ```

=== "C++"

    ```c++
    wallet->freeze_output(key_image);

    cout << wallet->is_output_frozen(key_image) << endl;    // 1

    wallet->thaw_output(key_image);
    ```

=== "C"

    ```c
    monero_wallet_freeze_output(wallet, key_image);

    bool frozen = false;
    monero_wallet_is_output_frozen(wallet, key_image, &frozen);    // true

    monero_wallet_thaw_output(wallet, key_image);
    ```
