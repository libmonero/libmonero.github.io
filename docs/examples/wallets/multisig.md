# Multisig wallets

A multisig wallet is shared by N participants, and M of them have to sign a transaction before it can be sent. In a 2-of-2 wallet both participants sign, and in a 2-of-3 wallet any two of the three do.

Each participant has its own wallet, and the participants trade plain-text strings to build the shared one. Only those strings travel, so the participants can be on different machines. The examples make a **2-of-2** wallet from `wallet_a` and `wallet_b`, and the same code works for any M and N.

!!! warning

    A multisig wallet holds real funds like any other, and a mistake in the setup can lock them. Develop on stagenet or testnet, and test the whole cycle with a small amount before you trust it with more.

## The wallets

Make one wallet for each participant. They are [full wallets](full.md), or wallets on wallet RPC servers: a server has one wallet open at a time, so two participants on RPC take two servers, on two ports. Do not use wallets that already have funds.

=== "Python"

    ```python
    from monero import *

    def make_participant(path: str) -> MoneroWalletFull:
        config = MoneroWalletConfig()
        config.path = path
        config.password = "supersecretpassword123"
        config.network_type = MoneroNetworkType.STAGENET
        config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
        return MoneroWalletFull.create_wallet(config)

    wallet_a = make_participant("multisig_a")
    wallet_b = make_participant("multisig_b")
    ```

=== "Java"

    ```java
    MoneroWalletFull makeParticipant(String path) {
      return MoneroWalletFull.createWallet(new MoneroWalletConfig()
              .setPath(path)
              .setPassword("supersecretpassword123")
              .setNetworkType(MoneroNetworkType.STAGENET)
              .setServerUri("http://localhost:38081")
              .setServerUsername("superuser")
              .setServerPassword("abctesting123"));
    }

    MoneroWalletFull walletA = makeParticipant("multisig_a");
    MoneroWalletFull walletB = makeParticipant("multisig_b");
    ```

=== "Kotlin"

    ```kotlin
    fun makeParticipant(path: String): MoneroWalletFull {
        val config = MoneroWalletConfig()
        config.path = path
        config.password = "supersecretpassword123"
        config.networkType = MoneroNetworkType.STAGENET
        config.server = MoneroRpcConnection("http://localhost:38081", "superuser", "abctesting123")
        return MoneroWalletFull.createWallet(config)
    }

    val walletA = makeParticipant("multisig_a")
    val walletB = makeParticipant("multisig_b")
    ```

=== "C#"

    ```csharp
    // two wallet RPC servers, one wallet open on each
    await using var walletA = new MoneroWalletRpc("http://localhost:38083", "rpc_user", "abc123");
    await walletA.CreateWalletAsync(new MoneroWalletConfig { Path = "multisig_a", Password = "supersecretpassword123" });

    await using var walletB = new MoneroWalletRpc("http://localhost:38084", "rpc_user", "abc123");
    await walletB.CreateWalletAsync(new MoneroWalletConfig { Path = "multisig_b", Password = "supersecretpassword123" });
    ```

=== "C++"

    ```c++
    using namespace monero;

    monero_wallet* make_participant(const string& path) {
      monero_wallet_config config;
      config.m_path = path;
      config.m_password = "supersecretpassword123";
      config.m_network_type = monero_network_type::STAGENET;
      config.m_server = make_shared<monero_rpc_connection>("http://localhost:38081", "superuser", "abctesting123");
      return monero_wallet_full::create_wallet(config);
    }

    monero_wallet* wallet_a = make_participant("multisig_a");
    monero_wallet* wallet_b = make_participant("multisig_b");
    ```

=== "C"

    ```c
    #include "monero_c.h"

    monero_wallet* wallet_a = NULL;
    monero_wallet_create_random("multisig_a", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET, "English", &wallet_a);
    monero_wallet_set_daemon_connection(wallet_a, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);

    monero_wallet* wallet_b = NULL;
    monero_wallet_create_random("multisig_b", "supersecretpassword123", MONERO_UTILS_NETWORK_STAGENET, "English", &wallet_b);
    monero_wallet_set_daemon_connection(wallet_b, "http://localhost:38081", "superuser", "abctesting123", "", NULL, true);
    ```

## Make the wallets multisig

Each participant prepares, and the prepared strings go to the others. Each participant then makes its wallet multisig with the strings of the others and the threshold. The participants exchange keys N - M + 1 times: once in a 2-of-2 wallet, and twice in a 2-of-3 one. After that the wallets share an address.

=== "Python"

    ```python
    password = "supersecretpassword123"
    wallets = [wallet_a, wallet_b]
    m = 2                                         # the threshold

    # each participant prepares, and the strings go to the others
    prepared = [wallet.prepare_multisig() for wallet in wallets]

    # each participant makes its wallet multisig with the strings of the others
    multisig_hexes = []
    for i, wallet in enumerate(wallets):
        others = [hex for j, hex in enumerate(prepared) if j != i]
        multisig_hexes.append(wallet.make_multisig(others, m, password))

    # exchange keys N - M + 1 times, each round with the strings of the last one
    for _ in range(len(wallets) - m + 1):
        results = [wallet.exchange_multisig_keys(multisig_hexes, password) for wallet in wallets]
        multisig_hexes = [result.multisig_hex for result in results]

    print(wallet_a.get_primary_address() == wallet_b.get_primary_address())   # True
    info = wallet_a.get_multisig_info()
    print(info.is_multisig, info.is_ready, info.threshold, info.num_participants)
    ```

=== "Java"

    ```java
    String password = "supersecretpassword123";
    List<MoneroWalletFull> wallets = Arrays.asList(walletA, walletB);
    int m = 2;                                    // the threshold

    // each participant prepares, and the strings go to the others
    List<String> prepared = new ArrayList<>();
    for (MoneroWalletFull wallet : wallets) prepared.add(wallet.prepareMultisig());

    // each participant makes its wallet multisig with the strings of the others
    List<String> multisigHexes = new ArrayList<>();
    for (int i = 0; i < wallets.size(); i++) {
      List<String> others = new ArrayList<>(prepared);
      others.remove(i);
      multisigHexes.add(wallets.get(i).makeMultisig(others, m, password));
    }

    // exchange keys N - M + 1 times, each round with the strings of the last one
    for (int round = 0; round < wallets.size() - m + 1; round++) {
      List<String> next = new ArrayList<>();
      for (MoneroWalletFull wallet : wallets) next.add(wallet.exchangeMultisigKeys(multisigHexes, password).getMultisigHex());
      multisigHexes = next;
    }

    System.out.println(walletA.getPrimaryAddress().equals(walletB.getPrimaryAddress()));   // true
    ```

=== "Kotlin"

    ```kotlin
    val password = "supersecretpassword123"
    val wallets = listOf(walletA, walletB)
    val m = 2                                     // the threshold

    // each participant prepares, and the strings go to the others
    val prepared = wallets.map { it.prepareMultisig() }

    // each participant makes its wallet multisig with the strings of the others
    var multisigHexes = wallets.mapIndexed { i, wallet ->
        wallet.makeMultisig(prepared.filterIndexed { j, _ -> j != i }, m, password)
    }

    // exchange keys N - M + 1 times, each round with the strings of the last one
    repeat(wallets.size - m + 1) {
        multisigHexes = wallets.map { it.exchangeMultisigKeys(multisigHexes, password).multisigHex!! }
    }

    println(walletA.getPrimaryAddress() == walletB.getPrimaryAddress())   // true
    ```

=== "C#"

    ```csharp
    string password = "supersecretpassword123";
    var wallets = new List<MoneroWallet> { walletA, walletB };
    int m = 2;                                    // the threshold

    // each participant prepares, and the strings go to the others
    var prepared = new List<string>();
    foreach (var wallet in wallets) prepared.Add(await wallet.PrepareMultisigAsync());

    // each participant makes its wallet multisig with the strings of the others
    var multisigHexes = new List<string>();
    for (int i = 0; i < wallets.Count; i++)
    {
        var others = prepared.Where((_, j) => j != i).ToList();
        multisigHexes.Add(await wallets[i].MakeMultisigAsync(others, m, password));
    }

    // exchange keys N - M + 1 times, each round with the strings of the last one
    for (int round = 0; round < wallets.Count - m + 1; round++)
    {
        var next = new List<string>();
        foreach (var wallet in wallets) next.Add((await wallet.ExchangeMultisigKeysAsync(multisigHexes, password)).MultisigHex!);
        multisigHexes = next;
    }

    Console.WriteLine(await walletA.GetPrimaryAddressAsync() == await walletB.GetPrimaryAddressAsync());   // True
    ```

=== "C++"

    ```c++
    string password = "supersecretpassword123";
    vector<monero_wallet*> wallets = { wallet_a, wallet_b };
    int m = 2;                                    // the threshold

    // each participant prepares, and the strings go to the others
    vector<string> prepared;
    for (monero_wallet* wallet : wallets) prepared.push_back(wallet->prepare_multisig());

    // each participant makes its wallet multisig with the strings of the others
    vector<string> multisig_hexes;
    for (size_t i = 0; i < wallets.size(); i++) {
      vector<string> others;
      for (size_t j = 0; j < prepared.size(); j++) if (j != i) others.push_back(prepared[j]);
      multisig_hexes.push_back(wallets[i]->make_multisig(others, m, password));
    }

    // exchange keys N - M + 1 times, each round with the strings of the last one
    for (size_t round = 0; round < wallets.size() - m + 1; round++) {
      vector<string> next;
      for (monero_wallet* wallet : wallets) next.push_back(wallet->exchange_multisig_keys(multisig_hexes, password).m_multisig_hex.get());
      multisig_hexes = next;
    }

    cout << (wallet_a->get_primary_address() == wallet_b->get_primary_address()) << endl;   // 1
    ```

=== "C"

    ```c
    const char* password = "supersecretpassword123";

    // each participant prepares, and the strings go to the others
    char* prepared_a = NULL;
    char* prepared_b = NULL;
    monero_wallet_prepare_multisig(wallet_a, &prepared_a);
    monero_wallet_prepare_multisig(wallet_b, &prepared_b);

    // each participant makes its wallet multisig with the strings of the others, with the threshold 2
    const char* for_a[] = { prepared_b };
    const char* for_b[] = { prepared_a };
    char* hex_a = NULL;
    char* hex_b = NULL;
    monero_wallet_make_multisig(wallet_a, for_a, 1, 2, password, &hex_a);
    monero_wallet_make_multisig(wallet_b, for_b, 1, 2, password, &hex_b);

    // exchange keys N - M + 1 = 1 time, with the strings of the other participant
    const char* exchange_a[] = { hex_b };
    const char* exchange_b[] = { hex_a };
    char* result_a = NULL;   // JSON, with the "multisigHex" that a third round would take
    char* result_b = NULL;
    monero_wallet_exchange_multisig_keys(wallet_a, exchange_a, 1, password, &result_a);
    monero_wallet_exchange_multisig_keys(wallet_b, exchange_b, 1, password, &result_b);
    ```

## Spend from a multisig wallet

A multisig wallet that received funds has to know the outputs that the other participants know. Each participant exports its multisig hex, and imports the ones of the others. Then one participant creates the transaction, the others sign it, and anyone submits it when enough of them have signed. In a 2-of-2 wallet, `wallet_b` is the only cosigner that is needed.

=== "Python"

    ```python
    # trade what each participant knows about the outputs
    hex_a = wallet_a.export_multisig_hex()
    hex_b = wallet_b.export_multisig_hex()
    wallet_a.import_multisig_hex([hex_b])
    wallet_b.import_multisig_hex([hex_a])

    # wallet_a creates the transaction
    tx_config = MoneroTxConfig()
    tx_config.account_index = 0
    tx_config.address = address
    tx_config.amount = 250000000000        # 0.25 XMR, in atomic units
    tx = wallet_a.create_tx(tx_config)
    multisig_tx_hex = tx.tx_set.multisig_tx_hex

    # wallet_b signs it, the threshold is reached
    sign_result = wallet_b.sign_multisig_tx_hex(multisig_tx_hex)

    # and the transaction is submitted
    tx_hashes = wallet_b.submit_multisig_tx_hex(sign_result.signed_multisig_tx_hex)
    print(tx_hashes[0])
    ```

=== "Java"

    ```java
    // trade what each participant knows about the outputs
    String hexA = walletA.exportMultisigHex();
    String hexB = walletB.exportMultisigHex();
    walletA.importMultisigHex(Arrays.asList(hexB));
    walletB.importMultisigHex(Arrays.asList(hexA));

    // walletA creates the transaction
    MoneroTxWallet tx = walletA.createTx(new MoneroTxConfig()
            .setAccountIndex(0)
            .setAddress(address)
            .setAmount("250000000000"));       // 0.25 XMR, in atomic units
    String multisigTxHex = tx.getTxSet().getMultisigTxHex();

    // walletB signs it, the threshold is reached
    MoneroMultisigSignResult signResult = walletB.signMultisigTxHex(multisigTxHex);

    // and the transaction is submitted
    List<String> txHashes = walletB.submitMultisigTxHex(signResult.getSignedMultisigTxHex());
    System.out.println(txHashes.get(0));
    ```

=== "Kotlin"

    ```kotlin
    // trade what each participant knows about the outputs
    val hexA = walletA.exportMultisigHex()
    val hexB = walletB.exportMultisigHex()
    walletA.importMultisigHex(listOf(hexB))
    walletB.importMultisigHex(listOf(hexA))

    // walletA creates the transaction
    val txConfig = MoneroTxConfig()
    txConfig.accountIndex = 0
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))   // 0.25 XMR, in atomic units
    val tx = walletA.createTxs(txConfig).first()
    val multisigTxHex = tx.txSet!!.multisigTxHex!!

    // walletB signs it, the threshold is reached
    val signResult = walletB.signMultisigTxHex(multisigTxHex)

    // and the transaction is submitted
    val txHashes = walletB.submitMultisigTxHex(signResult.signedMultisigTxHex!!)
    println(txHashes.first())
    ```

=== "C#"

    ```csharp
    // trade what each participant knows about the outputs
    string hexA = await walletA.ExportMultisigHexAsync();
    string hexB = await walletB.ExportMultisigHexAsync();
    await walletA.ImportMultisigHexAsync(new List<string> { hexB });
    await walletB.ImportMultisigHexAsync(new List<string> { hexA });

    // walletA creates the transaction
    var tx = await walletA.CreateTxAsync(new MoneroTxConfig
    {
        AccountIndex = 0,
        Address = address,
        Amount = 250000000000              // 0.25 XMR, in atomic units
    });
    string multisigTxHex = tx.TxSet!.MultisigTxHex!;

    // walletB signs it, the threshold is reached
    var signResult = await walletB.SignMultisigTxHexAsync(multisigTxHex);

    // and the transaction is submitted
    var txHashes = await walletB.SubmitMultisigTxHexAsync(signResult.SignedMultisigTxHex!);
    Console.WriteLine(txHashes[0]);
    ```

=== "C++"

    ```c++
    // trade what each participant knows about the outputs
    string hex_a = wallet_a->export_multisig_hex();
    string hex_b = wallet_b->export_multisig_hex();
    wallet_a->import_multisig_hex({ hex_b });
    wallet_b->import_multisig_hex({ hex_a });

    // wallet_a creates the transaction
    monero_tx_config tx_config;
    tx_config.m_account_index = 0;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;     // 0.25 XMR, in atomic units
    shared_ptr<monero_tx_wallet> tx = wallet_a->create_tx(tx_config);
    string multisig_tx_hex = tx->m_tx_set.get()->m_multisig_tx_hex.get();

    // wallet_b signs it, the threshold is reached
    monero_multisig_sign_result sign_result = wallet_b->sign_multisig_tx_hex(multisig_tx_hex);

    // and the transaction is submitted
    vector<string> tx_hashes = wallet_b->submit_multisig_tx_hex(sign_result.m_signed_multisig_tx_hex.get());
    cout << tx_hashes[0] << endl;
    monero_utils::free(tx);
    ```

=== "C"

    ```c
    // trade what each participant knows about the outputs
    char* hex_a = NULL;
    char* hex_b = NULL;
    monero_wallet_export_multisig_hex(wallet_a, &hex_a);
    monero_wallet_export_multisig_hex(wallet_b, &hex_b);
    const char* to_a[] = { hex_b };
    const char* to_b[] = { hex_a };
    int32_t num_imported = 0;
    monero_wallet_import_multisig_hex(wallet_a, to_a, 1, true, &num_imported);
    monero_wallet_import_multisig_hex(wallet_b, to_b, 1, true, &num_imported);

    // wallet_a creates the transaction
    char tx_config[256];
    snprintf(tx_config, sizeof(tx_config),
             "{\"accountIndex\":0,\"destinations\":[{\"address\":\"%s\",\"amount\":250000000000}]}", address);
    char* tx = NULL;
    monero_wallet_create_tx(wallet_a, tx_config, &tx);   // JSON, with the "multisigTxHex" of its tx set
    char* multisig_tx_hex = NULL;                         // set it to the "multisigTxHex" of `tx`

    // wallet_b signs it, the threshold is reached
    char* sign_result = NULL;
    monero_wallet_sign_multisig_tx_hex(wallet_b, multisig_tx_hex, &sign_result);   // JSON, with the "signedMultisigTxHex"
    char* signed_multisig_tx_hex = NULL;                                             // set it to the "signedMultisigTxHex" of `sign_result`

    // and the transaction is submitted
    char* tx_hashes = NULL;
    monero_wallet_submit_multisig_tx_hex(wallet_b, signed_multisig_tx_hex, &tx_hashes);   // JSON array of the hashes
    ```

In a wallet with more cosigners, each cosigner that is needed passes the result of the last one on: `sign_multisig_tx_hex()` takes the hex that the creator made for the first cosigner, and the signed hex of each cosigner for the next. The last one submits.

In C, the results are JSON: read the `multisigTxHex` and the `signedMultisigTxHex` with your JSON library, and pass them on as they are.
