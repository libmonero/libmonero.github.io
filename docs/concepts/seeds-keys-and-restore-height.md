# Seeds, keys and restore height

## The keys

A wallet is two secret keys, and what is derived from them.

* The **private spend key** is what spends funds. Whoever has it owns the wallet.
* The **private view key** lets whoever has it see the funds that the wallet receives. It does not let them spend the funds, and it does not tell them which of the outputs are spent.
* The **address** is made of the two public keys. It is safe to share, and it is what a payer needs.

A wallet that has the address and the private view key, but not the spend key, is **view-only**: it sees everything that arrives, and it cannot spend. It is the basis of the [view-only and offline](../examples/wallets/view-only-offline.md) setup, where the spend key stays on a machine that is never online. A wallet that has only the keys, and does not look at the chain, is a [keys wallet](../examples/wallets/keys.md).

## The seed

The **seed**, or mnemonic, is the spend key written as 25 words, in a language of your choice. It is the way to carry a wallet: from the seed, a wallet gets its spend key, its view key, and all its accounts and subaddresses. Anyone who has the seed has the funds, and no one without it can help you if you lose it.

* Write the seed on paper, and keep it where you keep other things of value. Do not keep it in a file on the machine of the wallet, a chat, or a photo.
* A seed can have an **offset**, a passphrase that changes the wallet that it makes. The same words and another offset give another wallet. Without the offset, the words give the wrong one.
* Check a seed that a person gives you, with the [utilities](../examples/utilities.md#check-a-seed-and-the-keys), before you use it.

A wallet that was made from keys has no seed. The wallet files and their password are another way to keep a wallet, but they depend on the program that made them, and the seed does not.

## The restore height

A wallet finds its funds by reading the chain. When it is created, it has nothing to find, so it can start from the top. When it is **restored** from a seed, it does not know when the wallet was made, and without help it would have to read the chain from the first block. The **restore height** is the block that it starts from.

* A new wallet starts from the current height by itself: set nothing.
* A wallet that is restored should have the height of the day it was created, or a little before, for safety. [Find the height of a date](../examples/wallets/sync.md#find-the-height-of-a-date) gives it.
* Too high a height misses the funds that arrived before it. Too low a height only costs time. If you do not know, use a low one.

See [Restore a wallet from a seed and sync it](../examples/wallets/full.md#restore-a-wallet-from-a-seed-and-sync-it).

## What to back up

| What | Back it up | Why |
|------|-----------|-----|
| the seed | yes, on paper, offline | it is the wallet |
| the password | yes, apart from the seed | it opens the wallet files |
| the wallet files | yes | they have the labels, the address book, the notes and the keys |
| the restore height | write it with the seed | it makes a restore fast |
| the private view key | only if you need a view-only wallet | it shows your funds to who has it |
