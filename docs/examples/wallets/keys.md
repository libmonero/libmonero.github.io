# Keys wallet

A keys wallet has the keys, the seed and the addresses of an account, and nothing else: no files, no chain, no daemon. It needs no network, so it works on a machine that is never online. It has no balance and no transactions, because it never looks at the chain.

!!! note

    Python, Kotlin, C++ and C have a keys wallet. Java and C# do not have one yet: use a [full wallet](full.md) or the [wallet RPC](rpc.md) there, and the addresses, the keys and the signatures work the same way.

## Create a keys wallet

Make a random one, restore one from a seed, or make one from its keys. A wallet made from the address and the private view key is view-only: it has no spend key. Add the private spend key to get a wallet that can spend.

=== "Python"

    ```python
    from monero import *

    # a random wallet
    config = MoneroWalletConfig()
    config.network_type = MoneroNetworkType.STAGENET
    wallet = MoneroWalletKeys.create_wallet_random(config)

    # from a seed
    config = MoneroWalletConfig()
    config.network_type = MoneroNetworkType.STAGENET
    config.seed = "hefty value scenic..."
    wallet = MoneroWalletKeys.create_wallet_from_seed(config)

    # from keys
    config = MoneroWalletConfig()
    config.network_type = MoneroNetworkType.STAGENET
    config.primary_address = primary_address
    config.private_view_key = private_view_key
    config.private_spend_key = private_spend_key      # leave it out for a view-only wallet
    wallet = MoneroWalletKeys.create_wallet_from_keys(config)
    ```

=== "Kotlin"

    ```kotlin
    import monero.daemon.model.MoneroNetworkType
    import monero.wallet.MoneroWalletKeys
    import monero.wallet.model.MoneroWalletConfig

    // a random wallet
    val random = MoneroWalletConfig()
    random.networkType = MoneroNetworkType.STAGENET
    val wallet = MoneroWalletKeys.createWalletRandom(random)

    // from a seed
    val fromSeed = MoneroWalletConfig()
    fromSeed.networkType = MoneroNetworkType.STAGENET
    fromSeed.seed = "hefty value scenic..."
    MoneroWalletKeys.createWalletFromSeed(fromSeed).use { seedWallet -> /* ... */ }

    // from keys
    val fromKeys = MoneroWalletConfig()
    fromKeys.networkType = MoneroNetworkType.STAGENET
    fromKeys.primaryAddress = primaryAddress
    fromKeys.privateViewKey = privateViewKey
    fromKeys.privateSpendKey = privateSpendKey        // leave it out for a view-only wallet
    MoneroWalletKeys.createWalletFromKeys(fromKeys).use { keysWallet -> /* ... */ }
    ```

=== "C++"

    ```c++
    using namespace monero;

    // a random wallet
    monero_wallet_config random;
    random.m_network_type = monero_network_type::STAGENET;
    monero_wallet_keys* wallet = monero_wallet_keys::create_wallet_random(random);

    // from a seed
    monero_wallet_config from_seed;
    from_seed.m_network_type = monero_network_type::STAGENET;
    from_seed.m_seed = "hefty value scenic...";
    monero_wallet_keys* seed_wallet = monero_wallet_keys::create_wallet_from_seed(from_seed);

    // from keys
    monero_wallet_config from_keys;
    from_keys.m_network_type = monero_network_type::STAGENET;
    from_keys.m_primary_address = primary_address;
    from_keys.m_private_view_key = private_view_key;
    from_keys.m_private_spend_key = private_spend_key;   // leave it out for a view-only wallet
    monero_wallet_keys* keys_wallet = monero_wallet_keys::create_wallet_from_keys(from_keys);

    delete keys_wallet;
    delete seed_wallet;
    delete wallet;
    ```

=== "C"

    ```c
    #include "monero_c.h"

    // a random wallet
    monero_wallet* wallet = NULL;
    monero_wallet_keys_create_random(MONERO_UTILS_NETWORK_STAGENET, "English", &wallet);

    // from a seed
    monero_wallet* seed_wallet = NULL;
    monero_wallet_keys_create_from_seed(MONERO_UTILS_NETWORK_STAGENET, "hefty value scenic...", "", "English", &seed_wallet);

    // from keys
    monero_wallet* keys_wallet = NULL;
    monero_wallet_keys_create_from_keys(MONERO_UTILS_NETWORK_STAGENET, primary_address, private_view_key,
                                        private_spend_key,   // "" for a view-only wallet
                                        "English", &keys_wallet);

    monero_wallet_free(keys_wallet);
    monero_wallet_free(seed_wallet);
    monero_wallet_free(wallet);
    ```

## Read the seed, the keys and the addresses

The wallet derives its addresses from its keys, so a subaddress is only a call away, and it is the same one that a full wallet of the same seed gives.

=== "Python"

    ```python
    print(wallet.get_seed())                  # the mnemonic phrase
    print(wallet.get_private_spend_key())
    print(wallet.get_private_view_key())

    print(wallet.get_primary_address())       # account 0, subaddress 0
    print(wallet.get_address(0, 1))           # account 0, subaddress 1
    print(wallet.get_address(1, 0))           # account 1, subaddress 0
    ```

=== "Kotlin"

    ```kotlin
    println(wallet.getSeed())                 // the mnemonic phrase
    println(wallet.getPrivateSpendKey())
    println(wallet.getPrivateViewKey())

    println(wallet.getPrimaryAddress())       // account 0, subaddress 0
    println(wallet.getAddress(0, 1))          // account 0, subaddress 1
    println(wallet.getAddress(1, 0))          // account 1, subaddress 0
    ```

=== "C++"

    ```c++
    cout << wallet->get_seed() << endl;                // the mnemonic phrase
    cout << wallet->get_private_spend_key() << endl;
    cout << wallet->get_private_view_key() << endl;

    cout << wallet->get_primary_address() << endl;     // account 0, subaddress 0
    cout << wallet->get_address(0, 1) << endl;         // account 0, subaddress 1
    cout << wallet->get_address(1, 0) << endl;         // account 1, subaddress 0
    ```

=== "C"

    ```c
    char* seed = NULL;
    monero_wallet_get_seed(wallet, &seed);                  // the mnemonic phrase
    monero_utils_free(seed);

    char* view_key = NULL;
    monero_wallet_get_private_view_key(wallet, &view_key);
    monero_utils_free(view_key);

    char* primary_address = NULL;
    monero_wallet_get_primary_address(wallet, &primary_address);   // account 0, subaddress 0
    monero_utils_free(primary_address);
    ```

A view-only wallet has no spend key and no seed, and asking it for them is an error.

## Sign and verify a message

Sign a message with the spend key, to prove that you own an address, and check a signature against an address. Verifying needs no key: any wallet does it.

=== "Python"

    ```python
    signature = wallet.sign_message("hello", MoneroMessageSignatureType.SIGN_WITH_SPEND_KEY, 0, 0)

    result = wallet.verify_message("hello", wallet.get_primary_address(), signature)
    print(result.is_good)                     # True
    ```

=== "Kotlin"

    ```kotlin
    val signature = wallet.signMessage("hello", MoneroMessageSignatureType.SIGN_WITH_SPEND_KEY, 0, 0)

    val result = wallet.verifyMessage("hello", wallet.getPrimaryAddress(), signature)
    println(result.isGood)                    // true
    ```

=== "C++"

    ```c++
    string signature = wallet->sign_message("hello", monero_message_signature_type::SIGN_WITH_SPEND_KEY, 0, 0);

    monero_message_signature_result result = wallet->verify_message("hello", wallet->get_primary_address(), signature);
    cout << result.m_is_good << endl;         // 1
    ```

=== "C"

    ```c
    char* signature = NULL;
    monero_wallet_sign_message(wallet, "hello", MONERO_MESSAGE_SIGN_WITH_SPEND_KEY, 0, 0, &signature);

    char* result = NULL;
    monero_wallet_verify_message(wallet, "hello", primary_address, signature, &result);   // JSON, with isGood

    monero_utils_free(result);
    monero_utils_free(signature);
    ```

A view-only wallet has no spend key, so it can only sign with the view key, with `SIGN_WITH_VIEW_KEY` instead of `SIGN_WITH_SPEND_KEY`.
