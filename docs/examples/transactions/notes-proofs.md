# Notes and proofs

Keep a note on a transaction, and prove a payment, a spend or the funds that a wallet holds. A proof is made by the wallet that has the funds, and checked by any wallet that has a daemon to look the transaction up.

## Keep a note on a transaction

A note is a text that the wallet keeps for a transaction, such as what it was for. It stays in the wallet, and it is never sent to the network.

=== "Python"

    ```python
    wallet.set_tx_note(tx_hash, "rent of November")
    print(wallet.get_tx_note(tx_hash))

    wallet.set_tx_notes([tx_hash_1, tx_hash_2], ["rent", "coffee"])      # several at once
    notes = wallet.get_tx_notes([tx_hash_1, tx_hash_2])
    ```

=== "Java"

    ```java
    wallet.setTxNote(txHash, "rent of November");
    System.out.println(wallet.getTxNote(txHash));

    wallet.setTxNotes(Arrays.asList(txHash1, txHash2), Arrays.asList("rent", "coffee"));   // several at once
    List<String> notes = wallet.getTxNotes(Arrays.asList(txHash1, txHash2));
    ```

=== "Kotlin"

    ```kotlin
    wallet.setTxNote(txHash, "rent of November")
    println(wallet.getTxNote(txHash))

    wallet.setTxNotes(listOf(txHash1, txHash2), listOf("rent", "coffee"))   // several at once
    val notes = wallet.getTxNotes(listOf(txHash1, txHash2))
    ```

=== "C#"

    ```csharp
    await wallet.SetTxNoteAsync(txHash, "rent of November");
    Console.WriteLine(await wallet.GetTxNoteAsync(txHash));

    await wallet.SetTxNotesAsync(new[] { txHash1, txHash2 }, new[] { "rent", "coffee" });   // several at once
    var notes = await wallet.GetTxNotesAsync(new[] { txHash1, txHash2 });
    ```

=== "C++"

    ```c++
    wallet->set_tx_note(tx_hash, "rent of November");
    cout << wallet->get_tx_note(tx_hash) << endl;

    wallet->set_tx_notes({ tx_hash_1, tx_hash_2 }, { "rent", "coffee" });   // several at once
    vector<string> notes = wallet->get_tx_notes({ tx_hash_1, tx_hash_2 });
    ```

=== "C"

    ```c
    monero_wallet_set_tx_note(wallet, tx_hash, "rent of November");

    char* note = NULL;
    monero_wallet_get_tx_note(wallet, tx_hash, &note);
    monero_utils_free(note);

    const char* hashes[] = { tx_hash_1, tx_hash_2 };
    const char* notes[] = { "rent", "coffee" };
    monero_wallet_set_tx_notes(wallet, hashes, notes, 2);   // several at once
    ```

## Prove a payment with the transaction key

The sender of a transaction can get its key. With the key, the hash and the address of the recipient, anyone can check that the transaction paid that address, and how much. It is the proof that you give to a recipient who says that they did not get the funds. Whoever has the key can show the payment to that address, so share it only with who has to see it.

=== "Python"

    ```python
    # the sender
    tx_key = wallet.get_tx_key(tx_hash)

    # whoever checks, with a wallet and a daemon
    check = other_wallet.check_tx_key(tx_hash, tx_key, address)

    print(check.is_good, check.received_amount)
    print(check.in_tx_pool, check.num_confirmations)
    ```

=== "Java"

    ```java
    // the sender
    String txKey = wallet.getTxKey(txHash);

    // whoever checks, with a wallet and a daemon
    MoneroCheckTx check = otherWallet.checkTxKey(txHash, txKey, address);

    System.out.println(check.isGood() + " " + check.getReceivedAmount());
    System.out.println(check.inTxPool() + " " + check.getNumConfirmations());
    ```

=== "Kotlin"

    ```kotlin
    // the sender
    val txKey = wallet.getTxKey(txHash)

    // whoever checks, with a wallet and a daemon
    val check = otherWallet.checkTxKey(txHash, txKey, address)

    println("${check.isGood} ${check.receivedAmount}")
    println("${check.inTxPool} ${check.numConfirmations}")
    ```

=== "C#"

    ```csharp
    // the sender
    string txKey = await wallet.GetTxKeyAsync(txHash);

    // whoever checks, with a wallet and a daemon
    var check = await otherWallet.CheckTxKeyAsync(txHash, txKey, address);

    Console.WriteLine($"{check.IsGood} {check.ReceivedAmount}");
    Console.WriteLine($"{check.InTxPool} {check.NumConfirmations}");
    ```

=== "C++"

    ```c++
    // the sender
    string tx_key = wallet->get_tx_key(tx_hash);

    // whoever checks, with a wallet and a daemon
    shared_ptr<monero_check_tx> check = other_wallet->check_tx_key(tx_hash, tx_key, address);

    cout << check->m_is_good << " " << check->m_received_amount.get() << endl;
    cout << check->m_in_tx_pool.get() << " " << check->m_num_confirmations.get() << endl;
    ```

=== "C"

    ```c
    // the sender
    char* tx_key = NULL;
    monero_wallet_get_tx_key(wallet, tx_hash, &tx_key);

    // whoever checks, with a wallet and a daemon
    char* check = NULL;
    monero_wallet_check_tx_key(other_wallet, tx_hash, tx_key, address, &check);   // JSON, with isGood, receivedAmount, inTxPool and numConfirmations
    printf("%s\n", check);

    monero_utils_free(check);
    monero_utils_free(tx_key);
    ```

## Prove a payment with a signature

A payment proof is a signature, in place of a key. The sender makes it for one transaction and one address, with a message of their choice, such as an invoice number. The checker gets the amount, and knows that the message is the one that was signed.

=== "Python"

    ```python
    # the sender
    signature = wallet.get_tx_proof(tx_hash, address, "invoice 42")

    # whoever checks
    check = other_wallet.check_tx_proof(tx_hash, address, "invoice 42", signature)

    print(check.is_good, check.received_amount, check.num_confirmations)
    ```

=== "Java"

    ```java
    // the sender
    String signature = wallet.getTxProof(txHash, address, "invoice 42");

    // whoever checks
    MoneroCheckTx check = otherWallet.checkTxProof(txHash, address, "invoice 42", signature);

    System.out.println(check.isGood() + " " + check.getReceivedAmount() + " " + check.getNumConfirmations());
    ```

=== "Kotlin"

    ```kotlin
    // the sender
    val signature = wallet.getTxProof(txHash, address, "invoice 42")

    // whoever checks
    val check = otherWallet.checkTxProof(txHash, address, "invoice 42", signature)

    println("${check.isGood} ${check.receivedAmount} ${check.numConfirmations}")
    ```

=== "C#"

    ```csharp
    // the sender
    string signature = await wallet.GetTxProofAsync(txHash, address, "invoice 42");

    // whoever checks
    var check = await otherWallet.CheckTxProofAsync(txHash, address, "invoice 42", signature);

    Console.WriteLine($"{check.IsGood} {check.ReceivedAmount} {check.NumConfirmations}");
    ```

=== "C++"

    ```c++
    // the sender
    string signature = wallet->get_tx_proof(tx_hash, address, "invoice 42");

    // whoever checks
    shared_ptr<monero_check_tx> check = other_wallet->check_tx_proof(tx_hash, address, "invoice 42", signature);

    cout << check->m_is_good << " " << check->m_received_amount.get() << " " << check->m_num_confirmations.get() << endl;
    ```

=== "C"

    ```c
    // the sender
    char* signature = NULL;
    monero_wallet_get_tx_proof(wallet, tx_hash, address, "invoice 42", &signature);

    // whoever checks
    char* check = NULL;
    monero_wallet_check_tx_proof(other_wallet, tx_hash, address, "invoice 42", signature, &check);   // JSON
    printf("%s\n", check);

    monero_utils_free(check);
    monero_utils_free(signature);
    ```

## Prove that you made a transaction

A spend proof shows that the wallet that signed it is the one that made the transaction, without telling who got the funds. It tells nothing about the recipient: use it to show that a transaction is yours.

=== "Python"

    ```python
    # the sender
    signature = wallet.get_spend_proof(tx_hash, "this is mine")

    # whoever checks
    print(other_wallet.check_spend_proof(tx_hash, "this is mine", signature))    # True
    ```

=== "Java"

    ```java
    // the sender
    String signature = wallet.getSpendProof(txHash, "this is mine");

    // whoever checks
    System.out.println(otherWallet.checkSpendProof(txHash, "this is mine", signature));    // true
    ```

=== "Kotlin"

    ```kotlin
    // the sender
    val signature = wallet.getSpendProof(txHash, "this is mine")

    // whoever checks
    println(otherWallet.checkSpendProof(txHash, "this is mine", signature))    // true
    ```

=== "C#"

    ```csharp
    // the sender
    string signature = await wallet.GetSpendProofAsync(txHash, "this is mine");

    // whoever checks
    Console.WriteLine(await otherWallet.CheckSpendProofAsync(txHash, "this is mine", signature));    // True
    ```

=== "C++"

    ```c++
    // the sender
    string signature = wallet->get_spend_proof(tx_hash, "this is mine");

    // whoever checks
    cout << other_wallet->check_spend_proof(tx_hash, "this is mine", signature) << endl;    // 1
    ```

=== "C"

    ```c
    // the sender
    char* signature = NULL;
    monero_wallet_get_spend_proof(wallet, tx_hash, "this is mine", &signature);

    // whoever checks
    bool good = false;
    monero_wallet_check_spend_proof(other_wallet, tx_hash, "this is mine", signature, &good);    // true

    monero_utils_free(signature);
    ```

## Prove that you hold funds

A reserve proof shows that a wallet holds funds, without moving them: the proof of an exchange that it has what it owes, for instance. Make one for the whole wallet, or for an amount of an account. The checker gets the total that the proof covers, and how much of it is already spent.

=== "Python"

    ```python
    # the whole wallet
    signature = wallet.get_reserve_proof_wallet("we hold it")

    # or at least an amount of an account
    signature = wallet.get_reserve_proof_account(0, 1000000000000, "we hold 1 XMR")

    # whoever checks
    check = other_wallet.check_reserve_proof(address, "we hold 1 XMR", signature)

    print(check.is_good, check.total_amount, check.unconfirmed_spent_amount)
    ```

=== "Java"

    ```java
    // the whole wallet
    String signature = wallet.getReserveProofWallet("we hold it");

    // or at least an amount of an account
    signature = wallet.getReserveProofAccount(0, new BigInteger("1000000000000"), "we hold 1 XMR");

    // whoever checks
    MoneroCheckReserve check = otherWallet.checkReserveProof(address, "we hold 1 XMR", signature);

    System.out.println(check.isGood() + " " + check.getTotalAmount() + " " + check.getUnconfirmedSpentAmount());
    ```

=== "Kotlin"

    ```kotlin
    // the whole wallet
    var signature = wallet.getReserveProofWallet("we hold it")

    // or at least an amount of an account
    signature = wallet.getReserveProofAccount(0, BigInteger("1000000000000"), "we hold 1 XMR")

    // whoever checks
    val check = otherWallet.checkReserveProof(address, "we hold 1 XMR", signature)

    println("${check.isGood} ${check.totalAmount} ${check.unconfirmedSpentAmount}")
    ```

=== "C#"

    ```csharp
    // the whole wallet
    string signature = await wallet.GetReserveProofWalletAsync("we hold it");

    // or at least an amount of an account
    signature = await wallet.GetReserveProofAccountAsync(0, 1000000000000, "we hold 1 XMR");

    // whoever checks
    var check = await otherWallet.CheckReserveProofAsync(address, "we hold 1 XMR", signature);

    Console.WriteLine($"{check.IsGood} {check.TotalAmount} {check.UnconfirmedSpentAmount}");
    ```

=== "C++"

    ```c++
    // the whole wallet
    string signature = wallet->get_reserve_proof_wallet("we hold it");

    // or at least an amount of an account
    signature = wallet->get_reserve_proof_account(0, 1000000000000, "we hold 1 XMR");

    // whoever checks
    shared_ptr<monero_check_reserve> check = other_wallet->check_reserve_proof(address, "we hold 1 XMR", signature);

    cout << check->m_is_good << " " << check->m_total_amount.get() << " " << check->m_unconfirmed_spent_amount.get() << endl;
    ```

=== "C"

    ```c
    // the whole wallet
    char* signature = NULL;
    monero_wallet_get_reserve_proof_wallet(wallet, "we hold it", &signature);
    monero_utils_free(signature);

    // or at least an amount of an account
    monero_wallet_get_reserve_proof_account(wallet, 0, 1000000000000, "we hold 1 XMR", &signature);

    // whoever checks
    char* check = NULL;
    monero_wallet_check_reserve_proof(other_wallet, address, "we hold 1 XMR", signature, &check);   // JSON, with isGood, totalAmount and unconfirmedSpentAmount
    printf("%s\n", check);

    monero_utils_free(check);
    monero_utils_free(signature);
    ```
