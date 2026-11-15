# Wallet files and attributes

A wallet that has files, a full wallet or a wallet on a server, keeps them at a path and encrypts them with a password. This page shows what a wallet tells about itself, how to change its password, and how to keep your own data in it. [Full wallet](full.md) has how to create, open, save and close it.

## Tell what the wallet is

The path of the wallet, the network that it is on, whether it is closed, and its public keys. The public keys are safe to share: they are part of the address. The private keys and the seed are in the [keys wallet](keys.md) examples.

!!! note

    monero-java does not have `get_network_type`.

=== "Python"

    ```python
    print(wallet.get_path())
    print(wallet.get_network_type())
    print(wallet.is_closed())

    print(wallet.get_public_view_key())
    print(wallet.get_public_spend_key())
    print(wallet.get_seed_language())
    ```

=== "Java"

    ```java
    System.out.println(wallet.getPath());
    System.out.println(wallet.isClosed());

    System.out.println(wallet.getPublicViewKey());
    System.out.println(wallet.getPublicSpendKey());
    System.out.println(wallet.getSeedLanguage());
    ```

=== "Kotlin"

    ```kotlin
    println(wallet.getPath())
    println(wallet.getNetworkType())
    println(wallet.isClosed())

    println(wallet.getPublicViewKey())
    println(wallet.getPublicSpendKey())
    println(wallet.getSeedLanguage())
    ```

=== "C#"

    ```csharp
    Console.WriteLine(await wallet.GetPathAsync());
    Console.WriteLine(await wallet.GetNetworkTypeAsync());
    Console.WriteLine(await wallet.IsClosedAsync());

    Console.WriteLine(await wallet.GetPublicViewKeyAsync());
    Console.WriteLine(await wallet.GetPublicSpendKeyAsync());
    Console.WriteLine(await wallet.GetSeedLanguageAsync());
    ```

=== "C++"

    ```c++
    cout << wallet->get_path() << endl;
    cout << (int) wallet->get_network_type() << endl;
    cout << wallet->is_closed() << endl;

    cout << wallet->get_public_view_key() << endl;
    cout << wallet->get_public_spend_key() << endl;
    cout << wallet->get_seed_language() << endl;
    ```

=== "C"

    ```c
    char* path = NULL;
    monero_wallet_get_path(wallet, &path);
    monero_utils_free(path);

    monero_utils_network_type network_type;
    monero_wallet_get_network_type(wallet, &network_type);

    bool closed = false;
    monero_wallet_is_closed(wallet, &closed);

    char* public_view_key = NULL;
    monero_wallet_get_public_view_key(wallet, &public_view_key);
    monero_utils_free(public_view_key);

    char* language = NULL;
    monero_wallet_get_seed_language(wallet, &language);
    monero_utils_free(language);
    ```

## Change the password

Change the password that encrypts the files. Give the old one and the new one, and save the wallet after the change.

=== "Python"

    ```python
    wallet.change_password("supersecretpassword123", "anothersecretpassword456")
    wallet.save()
    ```

=== "Java"

    ```java
    wallet.changePassword("supersecretpassword123", "anothersecretpassword456");
    wallet.save();
    ```

=== "Kotlin"

    ```kotlin
    wallet.changePassword("supersecretpassword123", "anothersecretpassword456")
    wallet.save()
    ```

=== "C#"

    ```csharp
    await wallet.ChangePasswordAsync("supersecretpassword123", "anothersecretpassword456");
    await wallet.SaveAsync();
    ```

=== "C++"

    ```c++
    wallet->change_password("supersecretpassword123", "anothersecretpassword456");
    wallet->save();
    ```

=== "C"

    ```c
    monero_wallet_change_password(wallet, "supersecretpassword123", "anothersecretpassword456");
    monero_wallet_save(wallet);
    ```

## Keep your own data in the wallet

A wallet keeps a few key and value pairs of text for you, in its files: an identifier of your own, a label, or the version of your application's data. Do not keep secrets in it that the wallet does not already protect, and do not count on it for much data.

=== "Python"

    ```python
    wallet.set_attribute("customer", "acme")

    print(wallet.get_attribute("customer"))      # acme
    ```

=== "Java"

    ```java
    wallet.setAttribute("customer", "acme");

    System.out.println(wallet.getAttribute("customer"));      // acme
    ```

=== "Kotlin"

    ```kotlin
    wallet.setAttribute("customer", "acme")

    println(wallet.getAttribute("customer"))      // acme
    ```

=== "C#"

    ```csharp
    await wallet.SetAttributeAsync("customer", "acme");

    Console.WriteLine(await wallet.GetAttributeAsync("customer"));      // acme
    ```

=== "C++"

    ```c++
    wallet->set_attribute("customer", "acme");

    string value;
    if (wallet->get_attribute("customer", value)) cout << value << endl;      // acme
    ```

=== "C"

    ```c
    monero_wallet_set_attribute(wallet, "customer", "acme");

    bool found = false;
    char* value = NULL;
    monero_wallet_get_attribute(wallet, "customer", &found, &value);
    if (found) printf("%s\n", value);      // acme
    monero_utils_free(value);
    ```

## Move the wallet

Move the files of a wallet from their path to another one, with the password that the wallet has there.

!!! note

    monero-java does not have `move_to`.

=== "Python"

    ```python
    wallet.move_to("/backup/sample_wallet_full", "supersecretpassword123")
    print(wallet.get_path())
    ```

=== "Kotlin"

    ```kotlin
    wallet.moveTo("/backup/sample_wallet_full", "supersecretpassword123")
    println(wallet.getPath())
    ```

=== "C#"

    ```csharp
    await wallet.MoveToAsync("/backup/sample_wallet_full", "supersecretpassword123");
    Console.WriteLine(await wallet.GetPathAsync());
    ```

=== "C++"

    ```c++
    wallet->move_to("/backup/sample_wallet_full", "supersecretpassword123");
    cout << wallet->get_path() << endl;
    ```

=== "C"

    ```c
    monero_wallet_move_to(wallet, "/backup/sample_wallet_full", "supersecretpassword123");
    ```
