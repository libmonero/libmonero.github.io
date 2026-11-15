# Errors

A call that fails does not return a value that you have to check: it fails with an error, in every language but C. The error says what went wrong, and an error of the RPC of a node or a wallet server also carries the code that the server gave. This page shows how to catch them, how to tell them apart, and how to check before you call, so that fewer of them happen.

## Catch an error

Wrap the calls that can fail, and look at the message. A call fails for many reasons: a server that does not answer, a wrong password, a wallet that is closed, an amount that is too high.

=== "Python"

    ```python
    from monero import *

    try:
        balance = wallet.get_balance()
    except (MoneroError, RuntimeError) as e:
        print("failed:", e)
    ```

=== "Java"

    ```java
    try {
      BigInteger balance = wallet.getBalance();
    } catch (MoneroError e) {
      System.out.println("failed: " + e.getMessage());
    }
    ```

=== "Kotlin"

    ```kotlin
    try {
        val balance = wallet.getBalance()
    } catch (e: MoneroError) {
        println("failed: ${e.message}")
    }
    ```

=== "C#"

    ```csharp
    try
    {
        ulong balance = await wallet.GetBalanceAsync();
    }
    catch (MoneroError e)
    {
        Console.WriteLine($"failed: {e.Message}");
    }
    ```

=== "C++"

    ```c++
    try {
      uint64_t balance = wallet->get_balance();
    } catch (const std::exception& e) {
      cout << "failed: " << e.what() << endl;
    }
    ```

=== "C"

    ```c
    uint64_t balance = 0;
    if (monero_wallet_get_balance(wallet, &balance) != MONERO_OK) {
      printf("failed: %s\n", monero_last_error());
    }
    ```

In C every call that can fail returns `MONERO_OK` or `MONERO_ERROR`, and the message of the last error is in `monero_last_error()`. The other examples of this site leave the checks out, to stay short, and you should not.

## Read the code of an RPC error

When a daemon or a wallet RPC server answers a call with an error, the error has the code that the server gave. For instance, a block that is higher than the chain does not exist. Catch the RPC error before the others to read its code.

=== "Python"

    ```python
    try:
        daemon.get_block_by_height(height + 1000000)
    except MoneroRpcError as e:
        print("rpc error", e.code, e)
    except (MoneroError, RuntimeError) as e:
        print("other error", e)
    ```

=== "Java"

    ```java
    try {
      daemon.getBlockByHeight(height + 1000000);
    } catch (MoneroRpcError e) {
      System.out.println("rpc error " + e.getCode() + " " + e.getRpcMethod() + " " + e.getMessage());
    } catch (MoneroError e) {
      System.out.println("other error " + e.getMessage());
    }
    ```

=== "Kotlin"

    ```kotlin
    try {
        daemon.getBlockByHeight(height + 1000000)
    } catch (e: MoneroRpcError) {
        println("rpc error ${e.code} ${e.message}")
    } catch (e: MoneroError) {
        println("other error ${e.message}")
    }
    ```

=== "C#"

    ```csharp
    try
    {
        await daemon.GetBlockByHeightAsync(height + 1000000);
    }
    catch (MoneroError e) when (e.Code == MoneroErrorCode.Rpc)
    {
        Console.WriteLine($"rpc error {e.Message}");
    }
    catch (MoneroError e)
    {
        Console.WriteLine($"other error {e.Message}");
    }
    ```

=== "C++"

    ```c++
    try {
      daemon->get_block_by_height(height + 1000000);
    } catch (const monero_rpc_error& e) {
      cout << "rpc error " << e.code << " " << e.what() << endl;
    } catch (const std::exception& e) {
      cout << "other error " << e.what() << endl;
    }
    ```

=== "C"

    ```c
    char* block = NULL;
    if (monero_daemon_get_block_by_height(daemon, height + 1000000, &block) != MONERO_OK) {
      if (monero_last_error_code() == MONERO_ERROR_RPC) {
        printf("rpc error %d %s\n", monero_last_error_rpc_code(), monero_last_error());
      } else {
        printf("other error %s\n", monero_last_error());
      }
    }
    monero_utils_free(block);
    ```

## The kinds of error

C and C# tell the kind of an error by a code. The other languages tell by the class of the exception, as above.

| Kind | C | C# | Meaning |
|------|---|----|---------|
| generic | `MONERO_ERROR_GENERIC` | `Generic` | any other failure |
| invalid argument | `MONERO_ERROR_INVALID_ARGUMENT` | `InvalidArgument` | a null value, a value out of range, or a text that does not parse |
| out of memory | `MONERO_ERROR_OUT_OF_MEMORY` | `OutOfMemory` | a memory allocation failed |
| RPC | `MONERO_ERROR_RPC` | `Rpc` | the daemon or the wallet RPC server answered with an error, or with an HTTP status that is not OK |
| I/O | `MONERO_ERROR_IO` | `Io` | the system reported an error of a file or a socket |

The kind comes from the type of the exception in monero-cpp, not from its text. A daemon that does not answer, a file that does not exist, an invalid seed and a wallet that is closed are all **generic**, so a program that has to tell them apart reads the message. Treat a kind that you do not know as generic: more can be added.

In C, `monero_last_error_code()` gives the kind, and `monero_last_error_rpc_code()` the code of an RPC error.

## Check before you call

Many errors are values that a program could have checked. Check them first, and the call that follows fails less:

* **An address or a seed that a person typed**: check it with the [utilities](utilities.md), before you make a transaction or a wallet from it.
* **A server that may be down**: [check the connection](rpc-connection.md#check-that-the-server-is-up) first, for instance when your program starts.
* **A wallet that has to be synced**: see [how far it is](wallets/sync.md#know-how-far-the-wallet-is) before you read its balance, since the balance of a wallet that has not synced is the one it had when it last stopped.
* **A transaction before it goes out**: [create it without relaying it](transactions/send.md), and look at the fee and the amounts, as a person would.

A node with a restricted RPC refuses the calls that change it, see [Daemon](daemon/index.md#restricted-and-unrestricted), and fails them with an RPC error.
