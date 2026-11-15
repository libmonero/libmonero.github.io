# Find transactions

Ask a wallet for its transactions and its transfers, by hash or by what they are, and read what they moved.

## Find a transaction by hash

Ask the wallet for the transactions that match a query. A query can match on more than the hash: it has a field for most of the properties of a transaction.

=== "Python"

    ```python
    query = MoneroTxQuery()
    query.hashes = [tx_hash]

    txs = wallet.get_txs(query)
    print(txs[0].fee, txs[0].is_confirmed)
    ```

=== "Java"

    ```java
    List<MoneroTxWallet> txs = wallet.getTxs(new MoneroTxQuery().setHashes(txHash));

    System.out.println(txs.get(0).getFee() + " " + txs.get(0).isConfirmed());
    ```

=== "Kotlin"

    ```kotlin
    val query = MoneroTxQuery()
    query.hashes = listOf(txHash)

    val txs = wallet.getTxs(query)
    println("${txs.first().fee} ${txs.first().isConfirmed}")
    ```

=== "C#"

    ```csharp
    var txs = await wallet.GetTxsAsync(new MoneroTxQuery { Hashes = new List<string> { txHash } });

    Console.WriteLine($"{txs[0].Fee} {txs[0].IsConfirmed}");
    ```

=== "C++"

    ```c++
    monero_tx_query query;
    query.m_hashes = { tx_hash };

    vector<shared_ptr<monero_tx_wallet>> txs = wallet->get_txs(query);
    cout << txs[0]->m_fee.get() << " " << txs[0]->m_is_confirmed.get() << endl;
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char query[256];
    snprintf(query, sizeof(query), "{\"hashes\":[\"%s\"]}", tx_hash);

    char* txs = NULL;
    monero_wallet_get_txs(wallet, query, &txs);   // JSON, the transactions in their blocks
    monero_utils_free(txs);
    ```

A transaction that the wallet does not have is not in the result.

## Find transactions by their properties

A query matches the transactions of a wallet by what they are: confirmed or in the pool, incoming or outgoing, and the heights they are in. A property that you leave out matches any value.

=== "Python"

    ```python
    query = MoneroTxQuery()
    query.is_confirmed = True
    query.is_incoming = True
    query.min_height = 573936
    query.max_height = 580000

    for tx in wallet.get_txs(query):
        print(tx.hash, tx.num_confirmations)
    ```

=== "Java"

    ```java
    MoneroTxQuery query = new MoneroTxQuery()
            .setIsConfirmed(true)
            .setIsIncoming(true)
            .setMinHeight(573936l)
            .setMaxHeight(580000l);

    for (MoneroTxWallet tx : wallet.getTxs(query)) {
      System.out.println(tx.getHash() + " " + tx.getNumConfirmations());
    }
    ```

=== "Kotlin"

    ```kotlin
    val query = MoneroTxQuery()
    query.isConfirmed = true
    query.isIncoming = true
    query.minHeight = 573936
    query.maxHeight = 580000

    for (tx in wallet.getTxs(query)) {
        println("${tx.hash} ${tx.numConfirmations}")
    }
    ```

=== "C#"

    ```csharp
    var query = new MoneroTxQuery
    {
        IsConfirmed = true,
        IsIncoming = true,
        MinHeight = 573936,
        MaxHeight = 580000
    };

    foreach (var tx in await wallet.GetTxsAsync(query))
    {
        Console.WriteLine($"{tx.Hash} {tx.NumConfirmations}");
    }
    ```

=== "C++"

    ```c++
    monero_tx_query query;
    query.m_is_confirmed = true;
    query.m_is_incoming = true;
    query.m_min_height = 573936;
    query.m_max_height = 580000;

    vector<shared_ptr<monero_tx_wallet>> txs = wallet->get_txs(query);
    for (const shared_ptr<monero_tx_wallet>& tx : txs) {
      cout << tx->m_hash.get() << " " << tx->m_num_confirmations.get() << endl;
    }
    monero_utils::free(txs);
    ```

=== "C"

    ```c
    char* txs = NULL;
    monero_wallet_get_txs(wallet, "{\"isConfirmed\":true,\"isIncoming\":true,\"minHeight\":573936,\"maxHeight\":580000}", &txs);   // JSON, the transactions in their blocks
    printf("%s\n", txs);
    monero_utils_free(txs);
    ```

## Read what a transaction moved

A transaction of a wallet knows the funds that came in and the funds that went out. An incoming transaction has one incoming transfer for each subaddress that it paid, and an outgoing one has a single outgoing transfer, with its destinations.

=== "Python"

    ```python
    for tx in wallet.get_txs(query):
        if tx.is_incoming:
            for transfer in tx.incoming_transfers:
                print("in", transfer.amount, transfer.account_index, transfer.subaddress_index)

        if tx.is_outgoing:
            transfer = tx.outgoing_transfer
            print("out", transfer.amount, tx.fee)
            for destination in transfer.destinations:
                print("  to", destination.address, destination.amount)
    ```

=== "Java"

    ```java
    for (MoneroTxWallet tx : wallet.getTxs(query)) {
      if (tx.isIncoming()) {
        for (MoneroIncomingTransfer transfer : tx.getIncomingTransfers()) {
          System.out.println("in " + transfer.getAmount() + " " + transfer.getAccountIndex() + " " + transfer.getSubaddressIndex());
        }
      }

      if (tx.isOutgoing()) {
        MoneroOutgoingTransfer transfer = tx.getOutgoingTransfer();
        System.out.println("out " + transfer.getAmount() + " " + tx.getFee());
        for (MoneroDestination destination : transfer.getDestinations()) {
          System.out.println("  to " + destination.getAddress() + " " + destination.getAmount());
        }
      }
    }
    ```

=== "Kotlin"

    ```kotlin
    for (tx in wallet.getTxs(query)) {
        if (tx.isIncoming == true) {
            for (transfer in tx.incomingTransfers.orEmpty()) {
                println("in ${transfer.amount} ${transfer.accountIndex} ${transfer.subaddressIndex}")
            }
        }

        if (tx.isOutgoing == true) {
            val transfer = tx.outgoingTransfer!!
            println("out ${transfer.amount} ${tx.fee}")
            for (destination in transfer.destinations.orEmpty()) {
                println("  to ${destination.address} ${destination.amount}")
            }
        }
    }
    ```

=== "C#"

    ```csharp
    foreach (var tx in await wallet.GetTxsAsync(query))
    {
        if (tx.IsIncoming == true)
        {
            foreach (var transfer in tx.IncomingTransfers ?? new())
            {
                Console.WriteLine($"in {transfer.Amount} {transfer.AccountIndex} {transfer.SubaddressIndex}");
            }
        }

        if (tx.IsOutgoing == true)
        {
            var transfer = tx.OutgoingTransfer!;
            Console.WriteLine($"out {transfer.Amount} {tx.Fee}");
            foreach (var destination in transfer.Destinations ?? new())
            {
                Console.WriteLine($"  to {destination.Address} {destination.Amount}");
            }
        }
    }
    ```

=== "C++"

    ```c++
    for (const shared_ptr<monero_tx_wallet>& tx : wallet->get_txs(query)) {
      if (tx->m_is_incoming.get()) {
        for (const shared_ptr<monero_incoming_transfer>& transfer : tx->m_incoming_transfers) {
          cout << "in " << transfer->m_amount.get() << " " << transfer->m_account_index.get() << " " << transfer->m_subaddress_index.get() << endl;
        }
      }

      if (tx->m_is_outgoing.get()) {
        shared_ptr<monero_outgoing_transfer> transfer = tx->m_outgoing_transfer;
        cout << "out " << transfer->m_amount.get() << " " << tx->m_fee.get() << endl;
        for (const shared_ptr<monero_destination>& destination : transfer->m_destinations) {
          cout << "  to " << destination->m_address.get() << " " << destination->m_amount.get() << endl;
        }
      }
    }
    ```

=== "C"

    ```c
    char* txs = NULL;
    monero_wallet_get_txs(wallet, NULL, &txs);   // JSON: each transaction has isIncoming and isOutgoing, its incomingTransfers
    printf("%s\n", txs);                          // with the amount, the account and the subaddress, and its outgoingTransfer
    monero_utils_free(txs);                       // with the amount and the destinations
    ```

## Get the transfers of a subaddress

The transfers of a wallet are the movements of funds, apart from the transactions that hold them. Ask for the ones of one subaddress, to see what it received and what it sent.

=== "Python"

    ```python
    query = MoneroTransferQuery()
    query.account_index = 0
    query.subaddress_index = 1

    for transfer in wallet.get_transfers(query):
        print(transfer.amount, transfer.tx.hash)
    ```

=== "Java"

    ```java
    MoneroTransferQuery query = new MoneroTransferQuery()
            .setAccountIndex(0)
            .setSubaddressIndex(1);

    for (MoneroTransfer transfer : wallet.getTransfers(query)) {
      System.out.println(transfer.getAmount() + " " + transfer.getTx().getHash());
    }
    ```

=== "Kotlin"

    ```kotlin
    val query = MoneroTransferQuery()
    query.accountIndex = 0
    query.subaddressIndex = 1

    for (transfer in wallet.getTransfers(query)) {
        println("${transfer.amount} ${transfer.tx?.hash}")
    }
    ```

=== "C#"

    ```csharp
    var query = new MoneroTransferQuery { AccountIndex = 0, SubaddressIndex = 1 };

    foreach (var transfer in await wallet.GetTransfersAsync(query))
    {
        Console.WriteLine($"{transfer.Amount} {transfer.Tx?.Hash}");
    }
    ```

=== "C++"

    ```c++
    monero_transfer_query query;
    query.m_account_index = 0;
    query.m_subaddress_index = 1;

    vector<shared_ptr<monero_transfer>> transfers = wallet->get_transfers(query);
    for (const shared_ptr<monero_transfer>& transfer : transfers) {
      cout << transfer->m_amount.get() << " " << transfer->m_tx->m_hash.get() << endl;
    }
    ```

=== "C"

    ```c
    char* transfers = NULL;
    monero_wallet_get_transfers(wallet, "{\"accountIndex\":0,\"subaddressIndex\":1}", &transfers);   // JSON, the transfers in their transactions
    printf("%s\n", transfers);
    monero_utils_free(transfers);
    ```
