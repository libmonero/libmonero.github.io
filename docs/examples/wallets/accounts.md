# Accounts and addresses

A wallet has accounts, and an account has subaddresses. Each subaddress is an address of its own that pays into the same wallet, so a wallet can give a different address to each payer, and tell which one paid. The calls on this page are the ones of every wallet implementation, except where a note says otherwise. See [Accounts and subaddresses](../../concepts/accounts-and-subaddresses.md) for how they are organized.

## List the accounts

Account 0 is the one that every wallet has. Its primary address is the address of the wallet, and each account has its own balance.

=== "Python"

    ```python
    for account in wallet.get_accounts():
        print(account.index, account.primary_address)
        print(account.balance, account.unlocked_balance)
    ```

=== "Java"

    ```java
    for (MoneroAccount account : wallet.getAccounts()) {
      System.out.println(account.getIndex() + " " + account.getPrimaryAddress());
      System.out.println(account.getBalance() + " " + account.getUnlockedBalance());
    }
    ```

=== "Kotlin"

    ```kotlin
    for (account in wallet.getAccounts()) {
        println("${account.index} ${account.primaryAddress}")
        println("${account.balance} ${account.unlockedBalance}")
    }
    ```

=== "C#"

    ```csharp
    foreach (var account in await wallet.GetAccountsAsync())
    {
        Console.WriteLine($"{account.Index} {account.PrimaryAddress}");
        Console.WriteLine($"{account.Balance} {account.UnlockedBalance}");
    }
    ```

=== "C++"

    ```c++
    for (const monero_account& account : wallet->get_accounts()) {
      cout << account.m_index.get() << " " << account.m_primary_address.get() << endl;
      cout << account.m_balance.get() << " " << account.m_unlocked_balance.get() << endl;
    }
    ```

=== "C"

    ```c
    char* accounts = NULL;
    monero_wallet_get_accounts(wallet, false, NULL, false, &accounts);   // JSON. false: no subaddresses, NULL: any tag, false: with the balances
    printf("%s\n", accounts);
    monero_utils_free(accounts);
    ```

## Create an account and a subaddress

An account groups addresses, such as the ones of a business, and a subaddress is one address for one payer or one invoice. The label is a text that the wallet keeps, to tell them apart.

=== "Python"

    ```python
    account = wallet.create_account("savings")
    print(account.index, account.primary_address)

    subaddress = wallet.create_subaddress(account.index, "invoice 42")
    print(subaddress.account_index, subaddress.index, subaddress.address)
    ```

=== "Java"

    ```java
    MoneroAccount account = wallet.createAccount("savings");
    System.out.println(account.getIndex() + " " + account.getPrimaryAddress());

    MoneroSubaddress subaddress = wallet.createSubaddress(account.getIndex(), "invoice 42");
    System.out.println(subaddress.getAccountIndex() + " " + subaddress.getIndex() + " " + subaddress.getAddress());
    ```

=== "Kotlin"

    ```kotlin
    val account = wallet.createAccount("savings")
    println("${account.index} ${account.primaryAddress}")

    val subaddress = wallet.createSubaddress(account.index!!, "invoice 42")
    println("${subaddress.accountIndex} ${subaddress.index} ${subaddress.address}")
    ```

=== "C#"

    ```csharp
    var account = await wallet.CreateAccountAsync("savings");
    Console.WriteLine($"{account.Index} {account.PrimaryAddress}");

    var subaddress = await wallet.CreateSubaddressAsync(account.Index!.Value, "invoice 42");
    Console.WriteLine($"{subaddress.AccountIndex} {subaddress.Index} {subaddress.Address}");
    ```

=== "C++"

    ```c++
    monero_account account = wallet->create_account("savings");
    cout << account.m_index.get() << " " << account.m_primary_address.get() << endl;

    monero_subaddress subaddress = wallet->create_subaddress(account.m_index.get(), "invoice 42");
    cout << subaddress.m_account_index.get() << " " << subaddress.m_index.get() << " " << subaddress.m_address.get() << endl;
    ```

=== "C"

    ```c
    char* account = NULL;
    monero_wallet_create_account(wallet, "savings", &account);              // JSON, with the index and the primary address
    monero_utils_free(account);

    char* subaddress = NULL;
    monero_wallet_create_subaddress(wallet, 1, "invoice 42", &subaddress);  // JSON, with the account index, the index and the address
    printf("%s\n", subaddress);
    monero_utils_free(subaddress);
    ```

## Look at the subaddresses of an account

The subaddresses of an account, with what each one holds, and whether it was used: a subaddress is used when it has received funds. The balance of one subaddress is also a call of its own.

=== "Python"

    ```python
    for subaddress in wallet.get_subaddresses(0):
        print(subaddress.index, subaddress.address, subaddress.label)
        print(subaddress.balance, subaddress.unlocked_balance)
        print(subaddress.num_unspent_outputs, subaddress.is_used)

    print(wallet.get_balance(0, 1))          # account 0, subaddress 1
    ```

=== "Java"

    ```java
    for (MoneroSubaddress subaddress : wallet.getSubaddresses(0)) {
      System.out.println(subaddress.getIndex() + " " + subaddress.getAddress() + " " + subaddress.getLabel());
      System.out.println(subaddress.getBalance() + " " + subaddress.getUnlockedBalance());
      System.out.println(subaddress.getNumUnspentOutputs() + " " + subaddress.isUsed());
    }

    System.out.println(wallet.getBalance(0, 1));          // account 0, subaddress 1
    ```

=== "Kotlin"

    ```kotlin
    for (subaddress in wallet.getSubaddresses(0)) {
        println("${subaddress.index} ${subaddress.address} ${subaddress.label}")
        println("${subaddress.balance} ${subaddress.unlockedBalance}")
        println("${subaddress.numUnspentOutputs} ${subaddress.isUsed}")
    }

    println(wallet.getBalance(0, 1))          // account 0, subaddress 1
    ```

=== "C#"

    ```csharp
    foreach (var subaddress in await wallet.GetSubaddressesAsync(0))
    {
        Console.WriteLine($"{subaddress.Index} {subaddress.Address} {subaddress.Label}");
        Console.WriteLine($"{subaddress.Balance} {subaddress.UnlockedBalance}");
        Console.WriteLine($"{subaddress.NumUnspentOutputs} {subaddress.IsUsed}");
    }

    Console.WriteLine(await wallet.GetBalanceAsync(0, 1));          // account 0, subaddress 1
    ```

=== "C++"

    ```c++
    for (const monero_subaddress& subaddress : wallet->get_subaddresses(0)) {
      cout << subaddress.m_index.get() << " " << subaddress.m_address.get() << " " << subaddress.m_label.get() << endl;
      cout << subaddress.m_balance.get() << " " << subaddress.m_unlocked_balance.get() << endl;
      cout << subaddress.m_num_unspent_outputs.get() << " " << subaddress.m_is_used.get() << endl;
    }

    cout << wallet->get_balance(0, 1) << endl;          // account 0, subaddress 1
    ```

=== "C"

    ```c
    char* subaddresses = NULL;
    monero_wallet_get_subaddresses(wallet, 0, NULL, 0, false, &subaddresses);   // JSON. NULL, 0: all of them, false: with the balances
    printf("%s\n", subaddresses);
    monero_utils_free(subaddresses);

    uint64_t balance = 0;
    monero_wallet_get_subaddress_balance(wallet, 0, 1, &balance);               // account 0, subaddress 1
    ```

## Find the subaddress of an address

Given an address that belongs to the wallet, get the account and the index that it is. It is how you tell which invoice a payment was for.

=== "Python"

    ```python
    subaddress = wallet.get_address_index(address)

    print(subaddress.account_index, subaddress.index)
    ```

=== "Java"

    ```java
    MoneroSubaddress subaddress = wallet.getAddressIndex(address);

    System.out.println(subaddress.getAccountIndex() + " " + subaddress.getIndex());
    ```

=== "Kotlin"

    ```kotlin
    val subaddress = wallet.getAddressIndex(address)

    println("${subaddress.accountIndex} ${subaddress.index}")
    ```

=== "C#"

    ```csharp
    var subaddress = await wallet.GetAddressIndexAsync(address);

    Console.WriteLine($"{subaddress.AccountIndex} {subaddress.Index}");
    ```

=== "C++"

    ```c++
    monero_subaddress subaddress = wallet->get_address_index(address);

    cout << subaddress.m_account_index.get() << " " << subaddress.m_index.get() << endl;
    ```

=== "C"

    ```c
    char* subaddress = NULL;
    monero_wallet_get_address_index(wallet, address, &subaddress);   // JSON, with the account index and the index
    printf("%s\n", subaddress);
    monero_utils_free(subaddress);
    ```

## Label a subaddress

Change the label of a subaddress, to say what it is for.

=== "Python"

    ```python
    wallet.set_subaddress_label(0, 1, "rent")
    ```

=== "Java"

    ```java
    wallet.setSubaddressLabel(0, 1, "rent");
    ```

=== "Kotlin"

    ```kotlin
    wallet.setSubaddressLabel(0, 1, "rent")
    ```

=== "C#"

    ```csharp
    await wallet.SetSubaddressLabelAsync(0, 1, "rent");
    ```

=== "C++"

    ```c++
    wallet->set_subaddress_label(0, 1, "rent");
    ```

=== "C"

    ```c
    monero_wallet_set_subaddress_label(wallet, 0, 1, "rent");
    ```

## Make an integrated address

An integrated address is an address with a payment id inside it. Whoever pays it sends the payment id with the funds, so you can tell the payers apart with a single address. Subaddresses do the same job better, and integrated addresses remain for services that still use them. Leave out the arguments to get one with a new random payment id.

=== "Python"

    ```python
    integrated = wallet.get_integrated_address()                       # a random payment id
    integrated = wallet.get_integrated_address(address, "03284e41c342f036")

    print(integrated.integrated_address, integrated.payment_id)

    decoded = wallet.decode_integrated_address(integrated.integrated_address)
    print(decoded.standard_address, decoded.payment_id)
    ```

=== "Java"

    ```java
    MoneroIntegratedAddress integrated = wallet.getIntegratedAddress();   // a random payment id
    integrated = wallet.getIntegratedAddress(address, "03284e41c342f036");

    System.out.println(integrated.getIntegratedAddress() + " " + integrated.getPaymentId());

    MoneroIntegratedAddress decoded = wallet.decodeIntegratedAddress(integrated.getIntegratedAddress());
    System.out.println(decoded.getStandardAddress() + " " + decoded.getPaymentId());
    ```

=== "Kotlin"

    ```kotlin
    var integrated = wallet.getIntegratedAddress()                       // a random payment id
    integrated = wallet.getIntegratedAddress(address, "03284e41c342f036")

    println("${integrated.integratedAddress} ${integrated.paymentId}")

    val decoded = wallet.decodeIntegratedAddress(integrated.integratedAddress!!)
    println("${decoded.standardAddress} ${decoded.paymentId}")
    ```

=== "C#"

    ```csharp
    var integrated = await wallet.GetIntegratedAddressAsync();                       // a random payment id
    integrated = await wallet.GetIntegratedAddressAsync(address, "03284e41c342f036");

    Console.WriteLine($"{integrated.IntegratedAddress} {integrated.PaymentId}");

    var decoded = await wallet.DecodeIntegratedAddressAsync(integrated.IntegratedAddress!);
    Console.WriteLine($"{decoded.StandardAddress} {decoded.PaymentId}");
    ```

=== "C++"

    ```c++
    monero_integrated_address integrated = wallet->get_integrated_address();   // a random payment id
    integrated = wallet->get_integrated_address(address, "03284e41c342f036");

    cout << integrated.m_integrated_address.get() << " " << integrated.m_payment_id.get() << endl;

    monero_integrated_address decoded = wallet->decode_integrated_address(integrated.m_integrated_address.get());
    cout << decoded.m_standard_address.get() << " " << decoded.m_payment_id.get() << endl;
    ```

=== "C"

    ```c
    char* integrated = NULL;
    monero_wallet_get_integrated_address(wallet, "", "03284e41c342f036", &integrated);   // JSON. "" as the address is the primary one
    printf("%s\n", integrated);
    monero_utils_free(integrated);
    ```

The [utilities](../utilities.md) make an integrated address from an address and a payment id with no wallet at all.

## Keep an address book

The address book is a list of addresses with a description, kept by the wallet, for the people that you pay often. An entry has an index that the wallet gives it when you add it.

=== "Python"

    ```python
    index = wallet.add_address_book_entry(address, "landlord")

    for entry in wallet.get_address_book_entries():
        print(entry.index, entry.address, entry.description)

    wallet.edit_address_book_entry(index, False, "", True, "landlord and rent")   # change the description only

    wallet.delete_address_book_entry(index)
    ```

=== "Java"

    ```java
    int index = wallet.addAddressBookEntry(address, "landlord");

    for (MoneroAddressBookEntry entry : wallet.getAddressBookEntries()) {
      System.out.println(entry.getIndex() + " " + entry.getAddress() + " " + entry.getDescription());
    }

    wallet.editAddressBookEntry(index, false, "", true, "landlord and rent");   // change the description only

    wallet.deleteAddressBookEntry(index);
    ```

=== "Kotlin"

    ```kotlin
    val index = wallet.addAddressBookEntry(address, "landlord")

    for (entry in wallet.getAddressBookEntries()) {
        println("${entry.index} ${entry.address} ${entry.description}")
    }

    wallet.editAddressBookEntry(index, null, "landlord and rent")   // change the description only

    wallet.deleteAddressBookEntry(index)
    ```

=== "C#"

    ```csharp
    ulong index = await wallet.AddAddressBookEntryAsync(address, "landlord");

    foreach (var entry in await wallet.GetAddressBookEntriesAsync())
    {
        Console.WriteLine($"{entry.Index} {entry.Address} {entry.Description}");
    }

    await wallet.EditAddressBookEntryAsync(index, null, "landlord and rent");   // change the description only

    await wallet.DeleteAddressBookEntryAsync(index);
    ```

=== "C++"

    ```c++
    uint64_t index = wallet->add_address_book_entry(address, "landlord");

    for (const monero_address_book_entry& entry : wallet->get_address_book_entries({})) {   // no indices: all the entries
      cout << entry.m_index.get() << " " << entry.m_address.get() << " " << entry.m_description.get() << endl;
    }

    wallet->edit_address_book_entry(index, false, "", true, "landlord and rent");   // change the description only

    wallet->delete_address_book_entry(index);
    ```

=== "C"

    ```c
    uint64_t index = 0;
    monero_wallet_add_address_book_entry(wallet, address, "landlord", &index);

    char* entries = NULL;
    monero_wallet_get_address_book_entries(wallet, NULL, 0, &entries);   // JSON. NULL, 0: all the entries
    printf("%s\n", entries);
    monero_utils_free(entries);

    monero_wallet_edit_address_book_entry(wallet, index, NULL, "landlord and rent");   // NULL: keep the address

    monero_wallet_delete_address_book_entry(wallet, index);
    ```

## Group accounts with tags

A tag groups accounts under a name, such as one for the accounts of a business. Tags are kept by a wallet RPC server, so these calls are for a [wallet on a server](rpc.md), and a native wallet does not support them.

=== "Python"

    ```python
    wallet.tag_accounts("business", [0, 1])
    wallet.set_account_tag_label("business", "Business accounts")

    for tag in wallet.get_account_tags():
        print(tag.tag, tag.label, tag.account_indices)

    wallet.untag_accounts([1])
    ```

=== "Java"

    ```java
    wallet.tagAccounts("business", Arrays.asList(0, 1));
    wallet.setAccountTagLabel("business", "Business accounts");

    for (MoneroAccountTag tag : wallet.getAccountTags()) {
      System.out.println(tag.getTag() + " " + tag.getLabel() + " " + tag.getAccountIndices());
    }

    wallet.untagAccounts(Arrays.asList(1));
    ```

=== "Kotlin"

    ```kotlin
    wallet.tagAccounts("business", listOf(0, 1))
    wallet.setAccountTagLabel("business", "Business accounts")

    for (tag in wallet.getAccountTags()) {
        println("${tag.tag} ${tag.label} ${tag.accountIndices}")
    }

    wallet.untagAccounts(listOf(1))
    ```

=== "C#"

    ```csharp
    await wallet.TagAccountsAsync("business", new uint[] { 0, 1 });
    await wallet.SetAccountTagLabelAsync("business", "Business accounts");

    foreach (var tag in await wallet.GetAccountTagsAsync())
    {
        Console.WriteLine($"{tag.Tag} {tag.Label} {string.Join(",", tag.AccountIndices!)}");
    }

    await wallet.UntagAccountsAsync(new uint[] { 1 });
    ```

=== "C++"

    ```c++
    wallet->tag_accounts("business", { 0, 1 });
    wallet->set_account_tag_label("business", "Business accounts");

    for (const shared_ptr<monero_account_tag>& tag : wallet->get_account_tags()) {
      cout << tag->m_tag.get() << " " << tag->m_label.get() << endl;
    }

    wallet->untag_accounts({ 1 });
    ```

=== "C"

    ```c
    uint32_t accounts[] = { 0, 1 };
    monero_wallet_tag_accounts(wallet, "business", accounts, 2);
    monero_wallet_set_account_tag_label(wallet, "business", "Business accounts");

    char* tags = NULL;
    monero_wallet_get_account_tags(wallet, &tags);   // JSON
    monero_utils_free(tags);

    uint32_t untag[] = { 1 };
    monero_wallet_untag_accounts(wallet, untag, 1);
    ```
