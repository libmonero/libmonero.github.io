# Send funds

Create a transaction from a wallet, look at what it costs, and relay it. Every example makes the transaction without relaying it first: the wallet gives the fee and the amounts, and nothing is sent until you relay it. They expect a `wallet` of any [type](../wallets/index.md) with unlocked funds, and an `address` to send to: on stagenet, use an address of that network.

## Create and relay a transfer

Create the transaction without relaying it, show the fee to the user, and relay it when they agree. The same code works for every wallet type: a wallet on a wallet RPC server and a native wallet create and relay a transaction the same way.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000  # 0.25 XMR, in atomic units
    tx_config.relay = False          # create it, do not relay it yet

    tx = wallet.create_tx(tx_config)
    fee = tx.fee  # "Are you sure you want to send... ?"

    wallet.relay_tx(tx)  # relay it
    ```

=== "Java"

    ```java
    MoneroTxWallet tx = wallet.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000")  // 0.25 XMR, in atomic units
            .setRelay(false));          // create it, do not relay it yet

    BigInteger fee = tx.getFee();  // "Are you sure you want to send... ?"

    wallet.relayTx(tx);  // relay it
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, amount))  // amount in atomic units
    txConfig.relay = false                                              // create it, do not relay it yet

    val tx = wallet.createTxs(txConfig).first()
    val fee = tx.fee  // "Are you sure you want to send... ?"

    val hash = wallet.relayTx(tx)  // relay it
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000,  // 0.25 XMR, in atomic units
        Relay = false           // create it, do not relay it yet
    };

    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    ulong? fee = tx.Fee;  // "Are you sure you want to send... ?"

    await wallet.RelayTxAsync(tx);  // relay it
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;  // 0.25 XMR, in atomic units
    tx_config.m_relay = false;          // create it, do not relay it yet

    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(tx_config);
    uint64_t fee = tx->m_fee.get();  // "Are you sure you want to send... ?"

    wallet->relay_tx(*tx);  // relay it
    monero_utils::free(tx);
    ```

=== "C"

    ```c
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}],\"relay\":false}", address);

    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);  // JSON, "Are you sure you want to send... ?"

    char* hash = NULL;
    monero_wallet_relay_tx_json(wallet, tx, &hash);  // relay it

    monero_utils_free(tx);
    monero_utils_free(hash);
    ```

A transaction takes about 10 blocks to unlock its outputs, so the change of a transfer is locked for a while and the unlocked balance is lower than the balance until then.

## Send to several recipients

One transaction can pay several addresses. Give a list of destinations in place of an address and an amount: each has its own, and the fee is added on top.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.destinations = [
        MoneroDestination(address_1, 250000000000),    # 0.25 XMR
        MoneroDestination(address_2, 100000000000),    # 0.1 XMR
    ]
    tx_config.relay = False

    tx = wallet.create_tx(tx_config)
    print(tx.fee)
    wallet.relay_tx(tx)
    ```

=== "Java"

    ```java
    MoneroTxWallet tx = wallet.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setDestinations(
                new MoneroDestination(address1, new BigInteger("250000000000")),    // 0.25 XMR
                new MoneroDestination(address2, new BigInteger("100000000000")))    // 0.1 XMR
            .setRelay(false));

    System.out.println(tx.getFee());
    wallet.relayTx(tx);
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(
        MoneroDestination(address1, BigInteger("250000000000")),    // 0.25 XMR
        MoneroDestination(address2, BigInteger("100000000000"))     // 0.1 XMR
    )
    txConfig.relay = false

    val tx = wallet.createTxs(txConfig).first()
    println(tx.fee)
    wallet.relayTx(tx)
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Destinations = new List<MoneroDestination>
        {
            new MoneroDestination(address1, 250000000000),    // 0.25 XMR
            new MoneroDestination(address2, 100000000000)     // 0.1 XMR
        },
        Relay = false
    };

    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    Console.WriteLine(tx.Fee);
    await wallet.RelayTxAsync(tx);
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_destinations = {
      make_shared<monero_destination>(address_1, 250000000000),    // 0.25 XMR
      make_shared<monero_destination>(address_2, 100000000000)     // 0.1 XMR
    };
    tx_config.m_relay = false;

    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(tx_config);
    cout << tx->m_fee.get() << endl;
    wallet->relay_tx(*tx);
    monero_utils::free(tx);
    ```

=== "C"

    ```c
    char tx_config[512];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000},"
             "{\"address\":\"%s\",\"amount\":100000000000}],\"relay\":false}", address_1, address_2);

    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);   // JSON
    char* hash = NULL;
    monero_wallet_relay_tx_json(wallet, tx, &hash);

    monero_utils_free(hash);
    monero_utils_free(tx);
    ```

## Choose the priority and the source

The priority sets how fast the transaction should be mined, and so what it pays: `UNIMPORTANT` the least, then `NORMAL`, then `ELEVATED`. `DEFAULT` is the one that the wallet picks. The account and the subaddresses choose which funds are spent: without them, the wallet picks from the account.

=== "Python"

    ```python
    print(wallet.get_default_fee_priority())             # what DEFAULT means for this wallet

    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.subaddress_indices = [1, 2]                # spend only from these subaddresses
    tx_config.address = address
    tx_config.amount = 250000000000
    tx_config.priority = MoneroTxPriority.ELEVATED
    tx_config.relay = False

    tx = wallet.create_tx(tx_config)
    ```

=== "Java"

    ```java
    System.out.println(wallet.getDefaultFeePriority());   // what DEFAULT means for this wallet

    MoneroTxWallet tx = wallet.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setSubaddressIndices(1, 2)                   // spend only from these subaddresses
            .setAddress(address)
            .setAmount("250000000000")
            .setPriority(MoneroTxPriority.ELEVATED)
            .setRelay(false));
    ```

=== "Kotlin"

    ```kotlin
    println(wallet.getDefaultFeePriority())               // what DEFAULT means for this wallet

    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.subaddressIndices = listOf(1, 2)             // spend only from these subaddresses
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))
    txConfig.priority = MoneroTxPriority.ELEVATED
    txConfig.relay = false

    val tx = wallet.createTxs(txConfig).first()
    ```

=== "C#"

    ```csharp
    Console.WriteLine(await wallet.GetDefaultFeePriorityAsync());   // what DEFAULT means for this wallet

    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        SubaddressIndices = new List<uint> { 1, 2 },      // spend only from these subaddresses
        Address = address,
        Amount = 250000000000,
        Priority = MoneroTxPriority.Elevated,
        Relay = false
    };

    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    ```

=== "C++"

    ```c++
    cout << wallet->get_default_fee_priority() << endl;    // what DEFAULT means for this wallet

    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_subaddress_indices = { 1, 2 };             // spend only from these subaddresses
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;
    tx_config.m_priority = monero_tx_priority::ELEVATED;
    tx_config.m_relay = false;

    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(tx_config);
    ```

=== "C"

    ```c
    monero_tx_priority priority;
    monero_wallet_get_default_fee_priority(wallet, &priority);   // what DEFAULT means for this wallet

    char tx_config[512];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"subaddressIndices\":[1,2],\"priority\":3,"
             "\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}],\"relay\":false}", address);   // 3 is ELEVATED

    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);   // JSON
    monero_utils_free(tx);
    ```

## Take the fee out of the amount

By default the fee is paid on top of the amount. To send everything that you give, with the fee taken out of it, name the destinations that pay it: the recipient then gets the amount minus its share of the fee.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000
    tx_config.subtract_fee_from = [0]      # the destination 0 pays the fee
    tx_config.relay = False

    tx = wallet.create_tx(tx_config)
    print(tx.fee, tx.output_sum)
    ```

=== "Java"

    ```java
    MoneroTxWallet tx = wallet.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000")
            .setSubtractFeeFrom(0)             // the destination 0 pays the fee
            .setRelay(false));

    System.out.println(tx.getFee() + " " + tx.getOutputSum());
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))
    txConfig.subtractFeeFrom = listOf(0)   // the destination 0 pays the fee
    txConfig.relay = false

    val tx = wallet.createTxs(txConfig).first()
    println("${tx.fee} ${tx.outputSum}")
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000,
        SubtractFeeFrom = new List<uint> { 0 },    // the destination 0 pays the fee
        Relay = false
    };

    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    Console.WriteLine($"{tx.Fee} {tx.OutputSum}");
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;
    tx_config.m_subtract_fee_from = { 0 };       // the destination 0 pays the fee
    tx_config.m_relay = false;

    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(tx_config);
    cout << tx->m_fee.get() << " " << tx->m_output_sum.get() << endl;
    ```

=== "C"

    ```c
    char tx_config[512];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"subtractFeeFrom\":[0],"
             "\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}],\"relay\":false}", address);   // the destination 0 pays the fee

    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);   // JSON
    monero_utils_free(tx);
    ```

## Split a transfer in several transactions

A transaction has a limit on its size, and a transfer from a wallet with many small outputs can need more than one. Allow the wallet to split it and it creates as many as it takes. `create_tx` makes exactly one and fails if the transfer does not fit, so use `create_txs` for a transfer that can split. Relay them together.

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000
    tx_config.can_split = True
    tx_config.relay = False

    txs = wallet.create_txs(tx_config)
    print(len(txs), sum(tx.fee for tx in txs))

    wallet.relay_txs(txs)
    ```

=== "Java"

    ```java
    List<MoneroTxWallet> txs = wallet.createTxs(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000")
            .setCanSplit(true)
            .setRelay(false));

    System.out.println(txs.size());

    wallet.relayTxs(txs);
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))
    txConfig.canSplit = true
    txConfig.relay = false

    val txs = wallet.createTxs(txConfig)
    println(txs.size)

    wallet.relayTxs(txs)
    ```

=== "C#"

    ```csharp
    var txConfig = new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000,
        CanSplit = true,
        Relay = false
    };

    var txs = await wallet.CreateTxsAsync(txConfig);
    Console.WriteLine(txs.Count);

    await wallet.RelayTxsAsync(txs);
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;
    tx_config.m_can_split = true;
    tx_config.m_relay = false;

    vector<shared_ptr<monero_tx_wallet>> txs = wallet->create_txs(tx_config);
    cout << txs.size() << endl;

    wallet->relay_txs(txs);
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char tx_config[512];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"canSplit\":true,"
             "\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}],\"relay\":false}", address);

    char* txs = NULL;
    monero_wallet_create_txs(wallet, tx_config, &txs);   // JSON, the transactions

    char* hashes = NULL;
    monero_wallet_relay_txs_json(wallet, txs, &hashes);  // JSON array of the hashes

    monero_utils_free(hashes);
    monero_utils_free(txs);
    ```

## Pay a payment request

A payment request is a `monero:` URI with an address and, optionally, an amount and a description: it is what a QR code of a payment holds. Parse it into a transaction config, and create the transaction from that.

=== "Python"

    ```python
    uri = "monero:" + address + "?tx_amount=0.25&tx_description=rent"

    tx_config = wallet.parse_payment_uri(uri)
    print(tx_config.address, tx_config.amount, tx_config.note)

    tx_config.relay = False
    tx = wallet.create_tx(tx_config)
    ```

=== "Java"

    ```java
    String uri = "monero:" + address + "?tx_amount=0.25&tx_description=rent";

    MoneroTxConfig txConfig = wallet.parsePaymentUri(uri);
    System.out.println(txConfig.getAddress() + " " + txConfig.getAmount() + " " + txConfig.getNote());

    txConfig.setRelay(false);
    MoneroTxWallet tx = wallet.createTx(txConfig);
    ```

=== "Kotlin"

    ```kotlin
    val uri = "monero:$address?tx_amount=0.25&tx_description=rent"

    val txConfig = wallet.parsePaymentUri(uri)
    println("${txConfig.address} ${txConfig.amount} ${txConfig.note}")

    txConfig.relay = false
    val tx = wallet.createTxs(txConfig).first()
    ```

=== "C#"

    ```csharp
    string uri = $"monero:{address}?tx_amount=0.25&tx_description=rent";

    var txConfig = await wallet.ParsePaymentUriAsync(uri);
    Console.WriteLine($"{txConfig.Address} {txConfig.Amount} {txConfig.Note}");

    txConfig.Relay = false;
    MoneroTxWallet tx = await wallet.CreateTxAsync(txConfig);
    ```

=== "C++"

    ```c++
    string uri = "monero:" + address + "?tx_amount=0.25&tx_description=rent";

    shared_ptr<monero_tx_config> tx_config = wallet->parse_payment_uri(uri);
    cout << tx_config->m_address.get() << " " << tx_config->m_amount.get() << endl;

    tx_config->m_relay = false;
    shared_ptr<monero_tx_wallet> tx = wallet->create_tx(*tx_config);
    ```

=== "C"

    ```c
    char uri[256];
    snprintf(uri, sizeof(uri), "monero:%s?tx_amount=0.25&tx_description=rent", address);

    char* tx_config = NULL;
    monero_wallet_parse_payment_uri(wallet, uri, &tx_config);   // JSON, a transaction config with the address, the amount and the note
    printf("%s\n", tx_config);

    // edit its "relay" to false if you need to, then
    char* tx = NULL;
    monero_wallet_create_tx(wallet, tx_config, &tx);

    monero_utils_free(tx);
    monero_utils_free(tx_config);
    ```

Check the amount and the address that the request gave before you send: they come from whoever made the request.
