# Utilities

The utilities are functions that need no daemon, no wallet and no network: they convert amounts, check addresses and seeds, and build payment requests. They are the same in every library, in a class or a namespace of their own: `MoneroUtils` in Python, Java and Kotlin, `monero_utils` in C++, and functions that start with `monero_utils_` in C.

!!! note

    monero-csharp does not have the utilities yet, and this page has no C# tab.

## Convert between XMR and atomic units

An amount is an integer in atomic units: 1 XMR is `1000000000000`. Keep amounts in atomic units in your program, and convert only to show them or to read what a person typed. The conversion uses a floating point number, so it is not meant for exact sums.

=== "Python"

    ```python
    from monero import *

    atomic = MoneroUtils.xmr_to_atomic_units(0.25)          # 250000000000
    xmr = MoneroUtils.atomic_units_to_xmr(250000000000)     # 0.25
    ```

=== "Java"

    ```java
    BigInteger atomic = MoneroUtils.xmrToAtomicUnits(0.25);              // 250000000000
    double xmr = MoneroUtils.atomicUnitsToXmr(new BigInteger("250000000000"));   // 0.25
    ```

=== "Kotlin"

    ```kotlin
    val atomic = MoneroUtils.xmrToAtomicUnits(0.25)                       // 250000000000
    val xmr = MoneroUtils.atomicUnitsToXmr(BigInteger("250000000000"))    // 0.25
    ```

=== "C++"

    ```c++
    uint64_t atomic = monero_utils::xmr_to_atomic_units(0.25);       // 250000000000
    double xmr = monero_utils::atomic_units_to_xmr(250000000000);    // 0.25
    ```

=== "C"

    ```c
    uint64_t atomic = 0;
    monero_utils_xmr_to_atomic_units(0.25, &atomic);                 // 250000000000

    double xmr = monero_utils_atomic_units_to_xmr(250000000000);     // 0.25
    ```

## Check an address

An address is valid for a network, and not for the others: a mainnet address is not good on stagenet. Check what a person typed before you use it. The `is_valid` form answers true or false, and the `validate` form fails with an error that says what is wrong.

=== "Python"

    ```python
    MoneroUtils.is_valid_address(address, MoneroNetworkType.STAGENET)     # True or False

    MoneroUtils.validate_address(address, MoneroNetworkType.STAGENET)     # raises MoneroError if it is not valid
    ```

=== "Java"

    ```java
    MoneroUtils.isValidAddress(address, MoneroNetworkType.STAGENET);      // true or false

    MoneroUtils.validateAddress(address, MoneroNetworkType.STAGENET);     // throws MoneroError if it is not valid
    ```

=== "Kotlin"

    ```kotlin
    MoneroUtils.isValidAddress(address, MoneroNetworkType.STAGENET)       // true or false

    MoneroUtils.validateAddress(address, MoneroNetworkType.STAGENET)      // throws MoneroError if it is not valid
    ```

=== "C++"

    ```c++
    monero_utils::is_valid_address(address, monero_network_type::STAGENET);    // true or false

    monero_utils::validate_address(address, monero_network_type::STAGENET);    // throws if it is not valid
    ```

=== "C"

    ```c
    bool valid = monero_utils_is_valid_address(address, MONERO_UTILS_NETWORK_STAGENET);   // true or false

    monero_result result = monero_utils_validate_address(address, MONERO_UTILS_NETWORK_STAGENET);   // MONERO_ERROR if it is not valid
    ```

## Check a seed and the keys

Check that a seed, or a key that someone gives you, is well formed, before you create a wallet from it. A seed has a language, and a check without one tries them all.

!!! note

    monero-java has `validateMnemonic`, which throws, and no `isValidMnemonic`.

=== "Python"

    ```python
    MoneroUtils.is_valid_mnemonic("hefty value scenic...")                  # True or False
    MoneroUtils.is_valid_mnemonic("hefty value scenic...", "English")

    MoneroUtils.is_valid_private_view_key(private_view_key)
    MoneroUtils.is_valid_private_spend_key(private_spend_key)
    MoneroUtils.is_valid_public_view_key(public_view_key)
    MoneroUtils.is_valid_public_spend_key(public_spend_key)
    ```

=== "Java"

    ```java
    MoneroUtils.validateMnemonic("hefty value scenic...");                 // throws MoneroError if it is not valid

    MoneroUtils.isValidPrivateViewKey(privateViewKey);
    MoneroUtils.isValidPrivateSpendKey(privateSpendKey);
    MoneroUtils.isValidPublicViewKey(publicViewKey);
    MoneroUtils.isValidPublicSpendKey(publicSpendKey);
    ```

=== "Kotlin"

    ```kotlin
    MoneroUtils.isValidMnemonic("hefty value scenic...")                   // true or false
    MoneroUtils.isValidMnemonic("hefty value scenic...", "English")

    MoneroUtils.isValidPrivateViewKey(privateViewKey)
    MoneroUtils.isValidPrivateSpendKey(privateSpendKey)
    MoneroUtils.isValidPublicViewKey(publicViewKey)
    MoneroUtils.isValidPublicSpendKey(publicSpendKey)
    ```

=== "C++"

    ```c++
    monero_utils::is_valid_mnemonic("hefty value scenic...");              // true or false
    monero_utils::is_valid_mnemonic("hefty value scenic...", "English");

    monero_utils::is_valid_private_view_key(private_view_key);
    monero_utils::is_valid_private_spend_key(private_spend_key);
    monero_utils::is_valid_public_view_key(public_view_key);
    monero_utils::is_valid_public_spend_key(public_spend_key);
    ```

=== "C"

    ```c
    monero_utils_is_valid_mnemonic("hefty value scenic...", "English");    // true or false

    monero_utils_is_valid_private_view_key(private_view_key);
    monero_utils_is_valid_private_spend_key(private_spend_key);
    monero_utils_is_valid_public_view_key(public_view_key);
    monero_utils_is_valid_public_spend_key(public_spend_key);
    ```

## Make an integrated address

An integrated address is a standard address with a payment id inside it. The utility makes one for a network, from an address and a payment id of 16 hexadecimal characters, with no wallet.

=== "Python"

    ```python
    integrated = MoneroUtils.get_integrated_address(MoneroNetworkType.STAGENET, address, "03284e41c342f036")

    print(integrated.integrated_address)
    print(integrated.standard_address, integrated.payment_id)

    MoneroUtils.is_valid_payment_id("03284e41c342f036")     # True
    ```

=== "Java"

    ```java
    MoneroIntegratedAddress integrated = MoneroUtils.getIntegratedAddress(MoneroNetworkType.STAGENET, address, "03284e41c342f036");

    System.out.println(integrated.getIntegratedAddress());
    System.out.println(integrated.getStandardAddress() + " " + integrated.getPaymentId());
    ```

=== "Kotlin"

    ```kotlin
    val integrated = MoneroUtils.getIntegratedAddress(MoneroNetworkType.STAGENET, address, "03284e41c342f036")

    println(integrated.integratedAddress)
    println("${integrated.standardAddress} ${integrated.paymentId}")

    MoneroUtils.isValidPaymentId("03284e41c342f036")     // true
    ```

=== "C++"

    ```c++
    monero_integrated_address integrated = monero_utils::get_integrated_address(monero_network_type::STAGENET, address, "03284e41c342f036");

    cout << integrated.m_integrated_address.get() << endl;
    cout << integrated.m_standard_address.get() << " " << integrated.m_payment_id.get() << endl;

    monero_utils::is_valid_payment_id("03284e41c342f036");     // true
    ```

=== "C"

    ```c
    char* integrated = NULL;
    monero_utils_get_integrated_address(MONERO_UTILS_NETWORK_STAGENET, address, "03284e41c342f036", &integrated);   // JSON
    printf("%s\n", integrated);
    monero_utils_free(integrated);

    monero_utils_is_valid_payment_id("03284e41c342f036");     // true
    ```

To read an integrated address, a wallet decodes it: see [Accounts and addresses](wallets/accounts.md#make-an-integrated-address).

## Make and read a payment request

A payment request is a `monero:` URI that holds an address and, optionally, an amount, a recipient name and a description. It is what a QR code of a payment shows. Build one from a transaction config, and read one back into a config.

!!! note

    monero-java builds a request with `getPaymentUri`, and reads one with `parsePaymentUri` of a [wallet](transactions/send.md#pay-a-payment-request).

=== "Python"

    ```python
    tx_config = MoneroTxConfig()
    tx_config.address = address
    tx_config.amount = 250000000000
    tx_config.recipient_name = "Acme"
    tx_config.note = "invoice 42"

    uri = MoneroUtils.get_payment_uri(tx_config, MoneroNetworkType.STAGENET)

    parsed = MoneroUtils.parse_payment_uri(uri, MoneroNetworkType.STAGENET)
    print(parsed.address, parsed.amount, parsed.recipient_name, parsed.note)
    ```

=== "Java"

    ```java
    MoneroTxConfig txConfig = new MoneroTxConfig()
            .setAddress(address)
            .setAmount("250000000000")
            .setRecipientName("Acme")
            .setNote("invoice 42");

    String uri = MoneroUtils.getPaymentUri(txConfig);
    ```

=== "Kotlin"

    ```kotlin
    val txConfig = MoneroTxConfig()
    txConfig.address = address
    txConfig.destinations = listOf(MoneroDestination(address, BigInteger("250000000000")))
    txConfig.recipientName = "Acme"
    txConfig.note = "invoice 42"

    val uri = MoneroUtils.getPaymentUri(txConfig, MoneroNetworkType.STAGENET)

    val parsed = MoneroUtils.parsePaymentUri(uri, MoneroNetworkType.STAGENET)
    println("${parsed.address} ${parsed.amount} ${parsed.recipientName} ${parsed.note}")
    ```

=== "C++"

    ```c++
    monero_tx_config tx_config;
    tx_config.m_address = address;
    tx_config.m_amount = 250000000000;
    tx_config.m_recipient_name = "Acme";
    tx_config.m_note = "invoice 42";

    string uri = monero_utils::get_payment_uri(tx_config, monero_network_type::STAGENET);

    shared_ptr<monero_tx_config> parsed = monero_utils::parse_payment_uri(uri, monero_network_type::STAGENET);
    cout << parsed->m_address.get() << " " << parsed->m_amount.get() << endl;
    ```

=== "C"

    ```c
    char* uri = NULL;
    monero_utils_get_payment_uri("{\"address\":\"...\",\"amount\":250000000000,\"recipientName\":\"Acme\",\"note\":\"invoice 42\"}",
                                 MONERO_UTILS_NETWORK_STAGENET, &uri);    // fill the address with yours

    char* parsed = NULL;
    monero_utils_parse_payment_uri(uri, MONERO_UTILS_NETWORK_STAGENET, &parsed);   // JSON, a transaction config

    monero_utils_free(parsed);
    monero_utils_free(uri);
    ```

## Set the log level

The libraries log what they do, from the native code of monero. Raise the level to see more when something goes wrong, and write the log to a file. The level goes from 0, the least, to 4, the most.

!!! note

    In monero-java the file is set with `configureNativeLogging(path, console)`, and it is `configure_logging` in the other languages.

=== "Python"

    ```python
    MoneroUtils.set_log_level(2)
    MoneroUtils.configure_logging("monero.log", False)     # to a file, and not to the console
    ```

=== "Java"

    ```java
    MoneroUtils.setLogLevel(2);
    MoneroUtils.configureNativeLogging("monero.log", false);     // to a file, and not to the console
    ```

=== "Kotlin"

    ```kotlin
    MoneroUtils.setLogLevel(2)
    MoneroUtils.configureLogging("monero.log", false)     // to a file, and not to the console
    ```

=== "C++"

    ```c++
    monero_utils::set_log_level(2);
    monero_utils::configure_logging("monero.log", false);     // to a file, and not to the console
    ```

=== "C"

    ```c
    monero_utils_set_log_level(2);
    monero_utils_configure_logging("monero.log", false);     // to a file, and not to the console
    ```
