# View-only and offline wallets

Two wallets split one account in two. The **view-only** wallet has the address and the private view key: it follows the chain, sees the funds, and builds a transaction, but it cannot sign one. The **offline** wallet has the spend key and never goes online: it signs what the view-only wallet builds. The spend key stays on a machine with no network.

The view-only wallet cannot see which outputs are spent, since that takes the spend key, so the two wallets trade what each one knows:

1. The view-only wallet exports its outputs, and the offline wallet imports them.
2. The offline wallet exports the key images of those outputs, and the view-only wallet imports them. Its balance is exact from there.
3. The view-only wallet creates an unsigned transaction.
4. The offline wallet signs it.
5. The view-only wallet submits the signed transaction.

Steps 1 and 2 are done once, and again when the wallet receives funds. The hex strings that the wallets trade are plain text, so they can go from one machine to the other on a USB drive, or as a QR code.

The wallets work with any implementation that can be made from keys. The examples use full wallets, and `wallet` is the wallet that owns the funds, from which the keys are taken. In C# they are two wallet RPC servers, and the offline one is started with `--offline`.

## Create the two wallets

The view-only wallet is made from the address and the private view key, with a daemon to sync with. The offline wallet is made from the keys with the spend key, and with no daemon: run it on a machine with no network. To keep it from reaching a daemon on a machine that has one, give it the address of a server that does not exist, as the tests of the libraries do.

=== "Python"

    ```python
    from monero import *

    # the view-only wallet: address and private view key, and a daemon
    config = MoneroWalletConfig()
    config.path = "view_only_wallet"
    config.password = "supersecretpassword123"
    config.network_type = MoneroNetworkType.STAGENET
    config.primary_address = wallet.get_primary_address()
    config.private_view_key = wallet.get_private_view_key()
    config.restore_height = 573936
    config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    view_only = MoneroWalletFull.create_wallet(config)
    view_only.sync()

    # the offline wallet: all the keys, and no daemon
    config = MoneroWalletConfig()
    config.path = "offline_wallet"
    config.password = "supersecretpassword123"
    config.network_type = MoneroNetworkType.STAGENET
    config.primary_address = wallet.get_primary_address()
    config.private_view_key = wallet.get_private_view_key()
    config.private_spend_key = wallet.get_private_spend_key()
    config.server = MoneroRpcConnection("offline_server_uri")
    offline = MoneroWalletFull.create_wallet(config)

    print(view_only.is_view_only(), offline.is_view_only())          # True False
    print(view_only.is_connected_to_daemon(), offline.is_connected_to_daemon())   # True False
    ```

=== "Java"

    ```java
    // the view-only wallet: address and private view key, and a daemon
    MoneroWalletFull viewOnly = MoneroWalletFull.createWallet(new MoneroWalletConfig()
            .setPath("view_only_wallet")
            .setPassword("supersecretpassword123")
            .setNetworkType(MoneroNetworkType.STAGENET)
            .setPrimaryAddress(wallet.getPrimaryAddress())
            .setPrivateViewKey(wallet.getPrivateViewKey())
            .setRestoreHeight(573936l)
            .setServerUri("http://localhost:38081")
            .setServerUsername("superuser")
            .setServerPassword("abctesting123"));
    viewOnly.sync();

    // the offline wallet: all the keys, and no daemon
    MoneroWalletFull offline = MoneroWalletFull.createWallet(new MoneroWalletConfig()
            .setPath("offline_wallet")
            .setPassword("supersecretpassword123")
            .setNetworkType(MoneroNetworkType.STAGENET)
            .setPrimaryAddress(wallet.getPrimaryAddress())
            .setPrivateViewKey(wallet.getPrivateViewKey())
            .setPrivateSpendKey(wallet.getPrivateSpendKey())
            .setServerUri("offline_server_uri"));

    System.out.println(viewOnly.isViewOnly() + " " + offline.isViewOnly());                       // true false
    System.out.println(viewOnly.isConnectedToDaemon() + " " + offline.isConnectedToDaemon());     // true false
    ```

=== "Kotlin"

    ```kotlin
    // the view-only wallet: address and private view key, and a daemon
    val viewOnlyConfig = MoneroWalletConfig()
    viewOnlyConfig.path = "view_only_wallet"
    viewOnlyConfig.password = "supersecretpassword123"
    viewOnlyConfig.networkType = MoneroNetworkType.STAGENET
    viewOnlyConfig.primaryAddress = wallet.getPrimaryAddress()
    viewOnlyConfig.privateViewKey = wallet.getPrivateViewKey()
    viewOnlyConfig.restoreHeight = 573936
    viewOnlyConfig.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
    val viewOnly = MoneroWalletFull.createWallet(viewOnlyConfig)
    viewOnly.sync()

    // the offline wallet: all the keys, and no daemon
    val offlineConfig = MoneroWalletConfig()
    offlineConfig.path = "offline_wallet"
    offlineConfig.password = "supersecretpassword123"
    offlineConfig.networkType = MoneroNetworkType.STAGENET
    offlineConfig.primaryAddress = wallet.getPrimaryAddress()
    offlineConfig.privateViewKey = wallet.getPrivateViewKey()
    offlineConfig.privateSpendKey = wallet.getPrivateSpendKey()
    offlineConfig.server = MoneroRpcConnection("offline_server_uri")
    val offline = MoneroWalletFull.createWallet(offlineConfig)

    println("${viewOnly.isViewOnly()} ${offline.isViewOnly()}")                         // true false
    println("${viewOnly.isConnectedToDaemon()} ${offline.isConnectedToDaemon()}")       // true false
    ```

=== "C#"

    ```csharp
    // two wallet RPC servers: the second one is started with --offline, and has no daemon
    await using var viewOnly = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");
    await viewOnly.CreateWalletAsync(new MoneroWalletConfig
    {
        Path = "view_only_wallet",
        Password = "supersecretpassword123",
        PrimaryAddress = await wallet.GetPrimaryAddressAsync(),
        PrivateViewKey = await wallet.GetPrivateViewKeyAsync(),
        RestoreHeight = 573936
    });

    await using var offline = new MoneroWalletRpc("http://localhost:38084", "rpc_user", "abc123");
    await offline.CreateWalletAsync(new MoneroWalletConfig
    {
        Path = "offline_wallet",
        Password = "supersecretpassword123",
        PrimaryAddress = await wallet.GetPrimaryAddressAsync(),
        PrivateViewKey = await wallet.GetPrivateViewKeyAsync(),
        PrivateSpendKey = await wallet.GetPrivateSpendKeyAsync()
    });
    ```

=== "C++"

    ```c++
    using namespace monero;

    // the view-only wallet: address and private view key, and a daemon
    monero_wallet_config view_only_config;
    view_only_config.m_path = "view_only_wallet";
    view_only_config.m_password = "supersecretpassword123";
    view_only_config.m_network_type = monero_network_type::STAGENET;
    view_only_config.m_primary_address = wallet->get_primary_address();
    view_only_config.m_private_view_key = wallet->get_private_view_key();
    view_only_config.m_restore_height = 573936;
    view_only_config.m_server = make_shared<monero_rpc_connection>("http://localhost:38081", "superuser", "abctesting123");
    monero_wallet* view_only = monero_wallet_full::create_wallet(view_only_config);
    view_only->sync();

    // the offline wallet: all the keys, and no daemon
    monero_wallet_config offline_config;
    offline_config.m_path = "offline_wallet";
    offline_config.m_password = "supersecretpassword123";
    offline_config.m_network_type = monero_network_type::STAGENET;
    offline_config.m_primary_address = wallet->get_primary_address();
    offline_config.m_private_view_key = wallet->get_private_view_key();
    offline_config.m_private_spend_key = wallet->get_private_spend_key();
    offline_config.m_server = make_shared<monero_rpc_connection>("offline_server_uri");
    monero_wallet* offline = monero_wallet_full::create_wallet(offline_config);
    ```

=== "C"

    ```c
    // the view-only wallet: address and private view key, and a daemon
    monero_wallet* view_only = NULL;
    monero_wallet_create_from_keys("view_only_wallet", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET,
                                   primary_address, private_view_key,
                                   "",                    // no private spend key
                                   573936, "English", &view_only);
    monero_wallet_set_daemon_connection(view_only, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);
    char* result = NULL;
    monero_wallet_sync(view_only, NULL, NULL, &result);
    monero_utils_free(result);

    // the offline wallet: all the keys, and no daemon
    monero_wallet* offline = NULL;
    monero_wallet_create_from_keys("offline_wallet", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET,
                                   primary_address, private_view_key, private_spend_key,
                                   0, "English", &offline);
    monero_wallet_set_daemon_connection(offline, "offline_server_uri", "", "", "", NULL, false);
    ```

## Create, sign and submit a transaction

`address` is where the funds go. The wallets trade strings only, and a hex string is all that the offline wallet needs to sign.

=== "Python"

    ```python
    # 1. the view-only wallet exports its outputs, the offline wallet imports them
    outputs_hex = view_only.export_outputs()
    offline.import_outputs(outputs_hex)

    # 2. the offline wallet exports the key images, the view-only wallet imports them
    key_images = offline.export_key_images()
    view_only.import_key_images(key_images.key_images)

    # 3. the view-only wallet creates an unsigned transaction
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000       # 0.25 XMR, in atomic units
    unsigned_tx = view_only.create_tx(tx_config)
    unsigned_tx_hex = unsigned_tx.tx_set.unsigned_tx_hex

    # 4. the offline wallet signs it
    signed_tx_set = offline.sign_txs(unsigned_tx_hex)

    # 5. the view-only wallet submits it
    tx_hashes = view_only.submit_txs(signed_tx_set.signed_tx_hex)
    print(tx_hashes[0])
    ```

=== "Java"

    ```java
    // 1. the view-only wallet exports its outputs, the offline wallet imports them
    String outputsHex = viewOnly.exportOutputs();
    offline.importOutputs(outputsHex);

    // 2. the offline wallet exports the key images, the view-only wallet imports them
    MoneroKeyImageExportResult keyImages = offline.exportKeyImages(true);
    viewOnly.importKeyImages(keyImages.getKeyImages());

    // 3. the view-only wallet creates an unsigned transaction
    MoneroTxWallet unsignedTx = viewOnly.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000"));       // 0.25 XMR, in atomic units
    String unsignedTxHex = unsignedTx.getTxSet().getUnsignedTxHex();

    // 4. the offline wallet signs it
    MoneroTxSet signedTxSet = offline.signTxs(unsignedTxHex);

    // 5. the view-only wallet submits it
    List<String> txHashes = viewOnly.submitTxs(signedTxSet.getSignedTxHex());
    System.out.println(txHashes.get(0));
    ```

=== "Kotlin"

    ```kotlin
    // 1. the view-only wallet exports its outputs, the offline wallet imports them
    val outputsHex = viewOnly.exportOutputs()
    offline.importOutputs(outputsHex)

    // 2. the offline wallet exports the key images, the view-only wallet imports them
    val keyImages = offline.exportKeyImages(true)
    viewOnly.importKeyImages(keyImages.keyImages!!)

    // 3. the view-only wallet creates an unsigned transaction
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))   // 0.25 XMR, in atomic units
    val unsignedTx = viewOnly.createTxs(txConfig).first()
    val unsignedTxHex = unsignedTx.txSet!!.unsignedTxHex!!

    // 4. the offline wallet signs it
    val signedTxSet = offline.signTxs(unsignedTxHex)

    // 5. the view-only wallet submits it
    val txHashes = viewOnly.submitTxs(signedTxSet.signedTxHex!!)
    println(txHashes.first())
    ```

=== "C#"

    ```csharp
    // 1. the view-only wallet exports its outputs, the offline wallet imports them
    string outputsHex = await viewOnly.ExportOutputsAsync();
    await offline.ImportOutputsAsync(outputsHex);

    // 2. the offline wallet exports the key images, the view-only wallet imports them
    var keyImages = await offline.ExportKeyImagesAsync(true);
    await viewOnly.ImportKeyImagesAsync(keyImages.KeyImages!);

    // 3. the view-only wallet creates an unsigned transaction
    var unsignedTx = await viewOnly.CreateTxAsync(new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000        // 0.25 XMR, in atomic units
    });
    string unsignedTxHex = unsignedTx.TxSet!.UnsignedTxHex!;

    // 4. the offline wallet signs it
    var signedTxSet = await offline.SignTxsAsync(unsignedTxHex);

    // 5. the view-only wallet submits it
    var txHashes = await viewOnly.SubmitTxsAsync(signedTxSet.SignedTxHex!);
    Console.WriteLine(txHashes[0]);
    ```

=== "C++"

    ```c++
    // 1. the view-only wallet exports its outputs, the offline wallet imports them
    string outputs_hex = view_only->export_outputs();
    offline->import_outputs(outputs_hex);

    // 2. the offline wallet exports the key images, the view-only wallet imports them
    shared_ptr<monero_key_image_export_result> key_images = offline->export_key_images(true);
    view_only->import_key_images(key_images->m_key_images);

    // 3. the view-only wallet creates an unsigned transaction
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;     // 0.25 XMR, in atomic units
    shared_ptr<monero_tx_wallet> unsigned_tx = view_only->create_tx(tx_config);
    string unsigned_tx_hex = unsigned_tx->m_tx_set.get()->m_unsigned_tx_hex.get();

    // 4. the offline wallet signs it
    monero_tx_set signed_tx_set = offline->sign_txs(unsigned_tx_hex);

    // 5. the view-only wallet submits it
    vector<string> tx_hashes = view_only->submit_txs(signed_tx_set.m_signed_tx_hex.get());
    cout << tx_hashes[0] << endl;
    monero_utils::free(unsigned_tx);
    ```

=== "C"

    ```c
    // 1. the view-only wallet exports its outputs, the offline wallet imports them
    char* outputs_hex = NULL;
    monero_wallet_export_outputs(view_only, false, &outputs_hex);
    int32_t num_imported = 0;
    monero_wallet_import_outputs(offline, outputs_hex, &num_imported);

    // 2. the offline wallet exports the key images, the view-only wallet imports them
    char* exported = NULL;
    monero_wallet_export_key_images(offline, true, &exported);   // JSON, with the "keyImages"
    char* key_images_json = NULL;                                 // set it to the "keyImages" array of `exported`, as a JSON array
    char* imported = NULL;
    monero_wallet_import_key_images(view_only, key_images_json, 0, &imported);

    // 3. the view-only wallet creates an unsigned transaction
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}]}", address);
    char* unsigned_tx = NULL;
    monero_wallet_create_tx(view_only, tx_config, &unsigned_tx);  // JSON, with the "unsignedTxHex" of its tx set
    char* unsigned_tx_hex = NULL;                                  // set it to the "unsignedTxHex" of `unsigned_tx`

    // 4. the offline wallet signs it
    char* signed_tx_set = NULL;
    monero_wallet_sign_txs(offline, unsigned_tx_hex, &signed_tx_set);   // JSON, with the "signedTxHex"
    char* signed_tx_hex = NULL;                                          // set it to the "signedTxHex" of `signed_tx_set`

    // 5. the view-only wallet submits it
    char* tx_hashes = NULL;
    monero_wallet_submit_txs(view_only, signed_tx_hex, &tx_hashes);     // JSON array of the hashes
    ```

In C, the results are JSON: read the `keyImages`, `unsignedTxHex` and `signedTxHex` with your JSON library, and pass them on as they are.

!!! warning

    The spend key and the seed are what the offline wallet keeps safe. Do not export them, do not paste them in a chat, and do not log them. A view-only wallet is not harmless either: it sees every transaction of the account, and so does anyone who gets its private view key.
