# Outputs and key images

What the chain knows about outputs, without knowing whose they are: if one is spent, how many there are, and where they are.

## Check if an output is spent

The key image of an output tells if it was spent: the node knows the key images of its chain and of its pool, without knowing whose they are. Get the key image of an output from a wallet.

=== "Python"

    ```python
    status = daemon.get_key_image_spent_status(key_image)

    if status == MoneroKeyImageSpentStatus.NOT_SPENT:
        print("not spent")
    elif status == MoneroKeyImageSpentStatus.CONFIRMED:
        print("spent in the chain")
    else:                                     # TX_POOL
        print("spent in a transaction of the pool")

    statuses = daemon.get_key_image_spent_statuses([key_image_1, key_image_2])   # several at once
    ```

=== "Java"

    ```java
    MoneroKeyImageSpentStatus status = daemon.getKeyImageSpentStatus(keyImage);

    if (status == MoneroKeyImageSpentStatus.NOT_SPENT) System.out.println("not spent");
    else if (status == MoneroKeyImageSpentStatus.CONFIRMED) System.out.println("spent in the chain");
    else System.out.println("spent in a transaction of the pool");   // TX_POOL

    List<MoneroKeyImageSpentStatus> statuses = daemon.getKeyImageSpentStatuses(Arrays.asList(keyImage1, keyImage2));   // several at once
    ```

=== "Kotlin"

    ```kotlin
    when (daemon.getKeyImageSpentStatus(keyImage)) {
        MoneroKeyImageSpentStatus.NOT_SPENT -> println("not spent")
        MoneroKeyImageSpentStatus.CONFIRMED -> println("spent in the chain")
        MoneroKeyImageSpentStatus.TX_POOL -> println("spent in a transaction of the pool")
    }

    val statuses = daemon.getKeyImageSpentStatuses(listOf(keyImage1, keyImage2))   // several at once
    ```

=== "C#"

    ```csharp
    var status = await daemon.GetKeyImageSpentStatusAsync(keyImage);

    switch (status)
    {
        case MoneroKeyImageSpentStatus.NotSpent: Console.WriteLine("not spent"); break;
        case MoneroKeyImageSpentStatus.Confirmed: Console.WriteLine("spent in the chain"); break;
        case MoneroKeyImageSpentStatus.TxPool: Console.WriteLine("spent in a transaction of the pool"); break;
    }

    var statuses = await daemon.GetKeyImageSpentStatusesAsync(new[] { keyImage1, keyImage2 });   // several at once
    ```

=== "C++"

    ```c++
    monero_key_image_spent_status status = daemon->get_key_image_spent_status(key_image);

    if (status == monero_key_image_spent_status::NOT_SPENT) cout << "not spent" << endl;
    else if (status == monero_key_image_spent_status::CONFIRMED) cout << "spent in the chain" << endl;
    else cout << "spent in a transaction of the pool" << endl;   // TX_POOL

    vector<monero_key_image_spent_status> statuses = daemon->get_key_image_spent_statuses({ key_image_1, key_image_2 });   // several at once
    ```

=== "C"

    ```c
    monero_key_image_spent_status status;
    monero_daemon_get_key_image_spent_status(daemon, key_image, &status);

    if (status == MONERO_KEY_IMAGE_NOT_SPENT) printf("not spent\n");
    else if (status == MONERO_KEY_IMAGE_CONFIRMED) printf("spent in the chain\n");
    else printf("spent in a transaction of the pool\n");   // MONERO_KEY_IMAGE_TX_POOL
    ```

## Get the histogram of the outputs

How many outputs of an amount the chain has. Since RingCT, the amount of an output is hidden and shown as `0`, so ask for that amount. The histogram of the older, visible amounts has an entry for each one of them.

=== "Python"

    ```python
    entries = daemon.get_output_histogram([0], None, None, None, None)

    for entry in entries:
        print(entry.amount, entry.num_instances)
    ```

=== "Java"

    ```java
    List<MoneroOutputHistogramEntry> entries = daemon.getOutputHistogram(Arrays.asList(BigInteger.ZERO), null, null, null, null);

    for (MoneroOutputHistogramEntry entry : entries) {
      System.out.println(entry.getAmount() + " " + entry.getNumInstances());
    }
    ```

=== "Kotlin"

    ```kotlin
    val entries = daemon.getOutputHistogram(listOf(BigInteger.ZERO))

    for (entry in entries) {
        println("${entry.amount} ${entry.numInstances}")
    }
    ```

=== "C#"

    ```csharp
    var entries = await daemon.GetOutputHistogramAsync(new ulong[] { 0 });

    foreach (var entry in entries)
    {
        Console.WriteLine($"{entry.Amount} {entry.NumInstances}");
    }
    ```

=== "C++"

    ```c++
    vector<shared_ptr<monero_output_histogram_entry>> entries =
        daemon->get_output_histogram({ 0 }, boost::none, boost::none, boost::none, boost::none);

    for (const shared_ptr<monero_output_histogram_entry>& entry : entries) {
      cout << entry->m_amount.get() << " " << entry->m_num_instances.get() << endl;
    }
    ```

=== "C"

    ```c
    uint64_t amounts[] = { 0 };

    char* entries = NULL;
    monero_daemon_get_output_histogram(daemon, amounts, 1, NULL, NULL, NULL, NULL, &entries);   // JSON
    printf("%s\n", entries);
    monero_utils_free(entries);
    ```

## Get the distribution of the outputs

For each height, how many outputs of an amount were created. A wallet uses it to pick the decoys of a ring. The distribution of a few blocks is short, and the one of the whole chain has an item for each block, so give a range.

=== "Python"

    ```python
    entries = daemon.get_output_distribution([0], False, height - 10, height - 1)

    for entry in entries:
        print(entry.amount, entry.start_height, entry.base, len(entry.distribution))
    ```

=== "Java"

    ```java
    // monero-java asks for the amounts only, and answers for the whole chain
    List<MoneroOutputDistributionEntry> entries = daemon.getOutputDistribution(Arrays.asList(BigInteger.ZERO));

    for (MoneroOutputDistributionEntry entry : entries) {
      System.out.println(entry.getAmount() + " " + entry.getStartHeight() + " " + entry.getBase() + " " + entry.getDistribution().size());
    }
    ```

=== "Kotlin"

    ```kotlin
    val entries = daemon.getOutputDistribution(listOf(BigInteger.ZERO), false, height - 10, height - 1)

    for (entry in entries) {
        println("${entry.amount} ${entry.startHeight} ${entry.base} ${entry.distribution?.size}")
    }
    ```

=== "C#"

    ```csharp
    var entries = await daemon.GetOutputDistributionAsync(new ulong[] { 0 }, false, height - 10, height - 1);

    foreach (var entry in entries)
    {
        Console.WriteLine($"{entry.Amount} {entry.StartHeight} {entry.Base} {entry.Distribution?.Count}");
    }
    ```

=== "C++"

    ```c++
    vector<shared_ptr<monero_output_distribution_entry>> entries =
        daemon->get_output_distribution({ 0 }, false, height - 10, height - 1);

    for (const shared_ptr<monero_output_distribution_entry>& entry : entries) {
      cout << entry->m_amount.get() << " " << entry->m_start_height.get() << " " << entry->m_distribution.size() << endl;
    }
    ```

=== "C"

    ```c
    uint64_t amounts[] = { 0 };
    bool is_cumulative = false;
    uint64_t start_height = height - 10;
    uint64_t end_height = height - 1;

    char* entries = NULL;
    monero_daemon_get_output_distribution(daemon, amounts, 1, &is_cumulative, &start_height, &end_height, &entries);   // JSON
    printf("%s\n", entries);
    monero_utils_free(entries);
    ```
