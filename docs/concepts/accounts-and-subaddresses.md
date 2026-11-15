# Accounts and subaddresses

A wallet does not have one address: it has as many as you ask for, and all of them pay into the same wallet. They are organized in two levels.

* An **account** is a group of addresses, with its own balance. Every wallet has account 0. Accounts are for who owns the funds: a business that keeps its accounts apart, or a person who keeps savings and spending.
* A **subaddress** is an address inside an account. It is identified by the index of its account and its own index, such as account 0, subaddress 3. Subaddress 0 of an account is its **primary address**, and the primary address of account 0 is the address of the wallet.

```text
wallet
├── account 0
│   ├── subaddress 0   primary address of the wallet
│   ├── subaddress 1   invoice 41
│   └── subaddress 2   invoice 42
└── account 1
    ├── subaddress 0   primary address of the account
    └── subaddress 1   ...
```

## Why use subaddresses

Give a different subaddress to each payer or each invoice. When funds arrive at one of them, you know who paid, since the [transfer](../examples/transactions/find.md) says the subaddress that it paid. Nobody else can tell that the subaddresses belong to the same wallet: an observer of the chain sees unrelated addresses, so one that paid you cannot see what another one paid.

A wallet watches all its subaddresses with one view key, so having many costs nothing in syncing. The older way to tell payers apart, the [integrated address](../examples/wallets/accounts.md#make-an-integrated-address) with a payment id, works too, but a subaddress is better in each aspect, and integrated addresses remain for services that already use them.

## Spending

A wallet spends from an account, and from the subaddresses of it that you choose, or from any of them if you do not choose. The funds of different accounts are not mixed in a transaction. See [Choose the priority and the source](../examples/transactions/send.md#choose-the-priority-and-the-source).

## Restoring

A wallet that is restored from a seed has the same accounts and subaddresses as the one that made the seed, since they are derived from the keys. The wallet only needs to find which ones were used, so it looks ahead of the last one that it knows. The lookahead is part of the wallet config, in `account_lookahead` and `subaddress_lookahead`. A subaddress is found again as long as it is within the lookahead, since it is derived from the keys.

The labels of accounts and subaddresses, the [address book](../examples/wallets/accounts.md#keep-an-address-book) and the notes on transactions are not derived from the keys, and a wallet restored from the seed does not have them. Back up the wallet files to keep them.

Read more: [Accounts and addresses](../examples/wallets/accounts.md).
