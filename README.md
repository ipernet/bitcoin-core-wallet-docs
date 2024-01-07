```
This documentation is experimental and may contain errors. Only use it for educational purpose.
```

# What is the documentation about?

> *Not your keys, not your coins*

**The documentation is about the only thing that matters for owning bitcoins on the long run: your seed key**.

**And so:**

- From a single **seed**
- We want to be able to **restore** all the **private keys**
- that will generate all the bitcoin **addresses**
- that can be part of a **legacy** or **modern** Bitcoin Core wallet.

**Where :**

> A **modern** Bitcoin wallet is:

A script-based wallet using SQLite as its [database backend](https://achow101.com/2020/10/0.21-wallets), supported since Bitcoin Core 0.23+ and named a [descriptor wallet](https://btctranscripts.com/advancing-bitcoin/2020/2020-02-06-andrew-chow-descriptor-wallets/).

> And a **Legacy wallet** is:

A static store of [private](https://learnmeabitcoin.com/beginners/keys_addresses) [keys](https://learnmeabitcoin.com/technical/private-key) (mostly), from where addresses are made, using the old Berkeley DB 4.8 database backend, created by default before Bitcoin Core 0.23.

> And [**addresses**](https://en.bitcoin.it/wiki/List_of_address_prefixes) are either:

- **Legacy addresses**
  - *P2PKH* / Pubkey hash / Legacy standard
  - Derivation path: `44'/0'/0'`
  - Start with **1**
- **Script hash addresses**
  - *P2SH* / Script hash / segwit
  - Derivation path: `49'/0'/0'`
  - Start with **2**
- **Native Segwit addresses**
  - *P2WPKH* / bech32 / native segwit / current standard
  - Derivation path: `84'/0'/0'`
  - Start with **bc1q**
- **Taproot addresses**
  - *P2TR* / bech32m / future standard
  - Derivation path: `86'/0'/0'`
  - Start with **bc1p**
  - Only supported with descriptor wallets.

> A **seed**:

- Is generated from [**at least 128 bits**](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#master-key-generation) of [**entropy**](https://learnmeabitcoin.com/technical/mnemonic#generate-entropy).
- Is between **128 and 512 bits** or **16 to 64 bytes** or **32 to 128** hexadecimal caracters.

---

> Again, if you are unfamiliar with the content detailed hereby, stick only to the standard way of using Bitcoin Core.

In both cases you'll need the Bitcoin Core client:

- **Download** the latest [Bitcoin Core client](https://bitcoincore.org/en/download/) on a secure computer.

> A *GNU/Linux x86-64 Fedora 34 workstation* is used here.

```bash
$ wget https://bitcoincore.org/bin/bitcoin-core-26.0/bitcoin-26.0-x86_64-linux-gnu.tar.gz
```

- **Verify** that the file you just downloaded is unaltered:

```bash
$ wget https://bitcoincore.org/bin/bitcoin-core-26.0/SHA256SUMS
$ sha256sum --ignore-missing --check SHA256SUMS
bitcoin-26.0-x86_64-linux-gnu.tar.gz: OK
```

- **Verify** that the binaries are signed:

https://github.com/bitcoin/bitcoin/tree/master/contrib/verify-binaries

> First, you have to figure out which public keys to recognize. Browse the [list of frequent builder-keys](https://github.com/bitcoin-core/guix.sigs/tree/main/builder-keys) and decide which of these keys you would like to trust. For each key you want to trust, you must obtain that key for your local GPG installation.


```
$ wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg
$ gpg --import achow101.gpg
gpg: key 17565732E08E5E41: 29 signatures not checked due to missing keys
gpg: key 17565732E08E5E41: public key "Andrew Chow <andrew@achow101.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no ultimately trusted keys found

# Or import all keys at once:
$ git clone https://github.com/bitcoin-core/guix.sigs
$ gpg --import guix.sigs/builder-keys/*

$ wget https://bitcoincore.org/bin/bitcoin-core-26.0/SHA256SUMS.asc
$ gpg --verify SHA256SUMS.asc
```
> Complete verify instructions: https://bitcoincore.org/en/download/


# Generating a seed

There are many ways of generating a seed, but it always starts with [generating enough entropy](https://learnmeabitcoin.com/technical/mnemonic#generate-entropy) as input.

## Generating entropy

A [BIP-32 Seed](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#master-key-generation) can be of length up to **512 bits**, using at least 128 bits of entropy.

For generating N bits of entropy, you can:
- Flip a coin N times and mark each result as *1* or *0*.
- Roll a dice a few hundreds of times (depending on the desired entropy amount).
- Get [entropy from a photograph](https://darkfield.dev/2021/01/30/entropy-from-images/).
- **Especially**, use a trusted (P)RNG.

> **[Seed entropy != Seed length](https://crypto.stackexchange.com/a/10405)**.

> About seed length, Bitcoin Core can only use seeds of max 256 bits length for **legacy** wallets (more on that later).


## Bitcoin Core and BIP39

BIP39 is a simple, human-friendly way to encode entropy into a mnemonic sentence that is easy to remember or archive, compared to the N bits of entropy you generated earlier.

> BIP39 is not about managing *user-generated sentences*. It is only about encoding entropy into a mnemonic.

The BIP39 processs generates **512-bits seeds** from **up to 256 bits of entropy** used as input. This default process is incompatible with the **256-bits seeds** required by Bitcoin Core for managing **legacy** wallets. Indeed, Bitcoin Core internally represents seeds for legacy wallets in a [32-bytes](https://github.com/bitcoin/bitcoin/issues/16393) data structure (and changing this is deemed as [not trivial](https://github.com/bitcoin/bitcoin/issues/19151#issuecomment-816755606)).

> Hence, the now legacy `sethdseed` RPC call of the Bitcoin Core client will only accept a 256-bits seed for creating/restoring a legacy wallet.

**As a consequence, you cannot use BIP39 to generate seeds for use with legacy wallets.**

> Note that the BIP39 mnemonic standard may have [some flaws](https://en.bitcoin.it/wiki/Seed_phrase#BIP39_and_its_flaws) and is not planned to be implemented in Core anyway (see [#17748](https://github.com/bitcoin/bitcoin/issues/17748), [#19151](https://github.com/bitcoin/bitcoin/issues/19151)).

However, with *descriptor* wallets, usage of 512-bits seeds is possible, thus making the use of BIP39 possible for managing the memorization and archiving of such seeds.

The [BIP39 process](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki):

- Starts with [generating](https://learnmeabitcoin.com/technical/mnemonic#generate-entropy) **128 to 256 bits** of **entropy** from a trusted source.
- Adds a checksum of the generated entropy.
- [Encode](https://learnmeabitcoin.com/technical/mnemonic#entropy-to-mnemonic) the generated entropy as a sorted list of 12 to 24-words (128/256 bits).

This "mnemonic" becomes **the only secret you need to remember** for managing your wallet on the long run (given it remains the sole seed used by your wallet in the long run).

**From the mnemonic, a root seed will be created as follow:**

- First, an optional *passphrase* must be chosen (default is to use an empty string)

> Setting a passphrase provides a 2 factor authentication process when restoring your wallet: The seed will be the *something you have* while the passphrase the *something you know*. Both needs to be remembered because any different passphrase would change the final seed to be generated.

- [The mnemonic and the optional passphrase](https://learnmeabitcoin.com/technical/mnemonic#mnemonic-to-seed) are sent by default for 2048 iterations to a slow **Key Derivation Function**: PBKDF2 (with HMAC-SHA512 as the hashing algorithm).

> This prevents brute force attack on the mnemonic and ensure a **512-bits output**.

The output of this process is a **BIP39 seed**.

**Example of a BIP-39 seed, in hex:**

> *db29b8749575cc9694754584a88ea8e2ffe5530ea2e22d500944384d7073e3e65521b6c0019817b060541a4b2dfee6a065c1f5245e702f29f9e5fe6c231183c7*

**Play with BIP-39**: https://iancoleman.io/bip39/

# From a seed to an Extended Master Private Key

Let's consider that we have a **512-bits seed number** (generated using BIP39 or not).

**Now comes [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki):**

To work down to individual private keys and generated addresses in a Bitcoin core wallet (legacy or not), Bitcoin Core will **hash** this seed with **HMAC-SHA512** and the "`Bitcoin seed`" string as the HMAC key.

This process also yields **[512 bits of output](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#master-key-generation)** where the left 256 bits become the [**Master Private Key**](https://developer.bitcoin.org/devguide/wallets.html#id6) while the right 256 bits become the [**Master Chain Code**](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#master-key-generation) of the BIP32 HD wallet. The chain code is **necessary** to derive child keys.

The **Master Extended Private Key** is the **Master Chain Code** and the **Master Private Key** [put together](https://developer.bitcoin.org/devguide/wallets.html#hierarchical-deterministic-key-creation). It is also named the **BIP32 Root Key**. They are the fundation of hierarchical deterministic wallets.

> A given seed will **always yield** the same root key when hashed with the very same process and settings.

> BIP32 root keys are usually stored in a [serialized](https://learnmeabitcoin.com/technical/extended-keys) format, starting with `xprv` (mainnet) or `tprv` (testnet).

The **Master Extended Private Key**, alongside a **[derivation path](https://learnmeabitcoin.com/technical/derivation-paths)** indicating the address standard to use are the starting point for generating all the **addresses** of your wallet.

### Recap

- **With legacy wallets:**
  - You cannot use BIP39 to generate a seed: a 512 bits output is incompatible.
  - You need to remember/archive a 256-bits seed, usually in [WIF format](https://learnmeabitcoin.com/technical/wif).
  - You should migrate the wallet to a descriptor wallet (to support taproot and the [long term](https://github.com/bitcoin/bitcoin/issues/20160)).

- **With descriptor wallets:**
  - You can use BIP39 and get a 24-words mnemonic for memorizing/archiving entropy up to 256 bits. If you stick to the unaltered BIP39 process, the mnemonic will always yield the same `BIP32 root key` that will be usable for generating whathever BIP 44/49/84/86 HD wallets standard you need to have in your wallet.
  - If you don't want to use BIP39, you still have 256 or 512 bits of entropy to remember some way.

## Import notes about wallets backups

Wallets managed with Bitcoin Core are living organisms, especially on the long run. For unstance, an original `hdseeds` can be rotated within the same legacy wallets, or new descriptors can be added with different root keys, or addresses can be generated with labels... As such, frequent backups of the full wallet file are still required because remembering a single seed will always mean only saving parts of what can make a complete wallet.

# Demo

> **Client**: Bitcoin Core 0.26

## Legacy wallet (won't work after 0.26)

- **Create a non-blank legacy wallet**:

```bash
$ ./bitcoind -deprecatedrpc=create_bdb -noconnect

$ ./bitcoin-cli -named createwallet wallet_name="legacy-wallet" blank=false passphrase="my-passphrase" descriptors=false load_on_startup=false
```
```json
{
  "name": "legacy-wallet",
  "warnings": [
    "Wallet created successfully. The legacy wallet type is being deprecated and support for creating and opening legacy wallets will be removed in the future."
  ]
}
```

- **Unlock the wallet for a 60 seconds**:

```
./bitcoin-cli walletpassphrase "my-passphrase" 60
```

- **Dump the legacy wallet**:

```
./bitcoin-cli dumpwallet "legacy-wallet.dump"
{
  "filename": "/home/user/legacy-wallet.dump"
}
```

- **Retrieve its generated hdseed and Extented Master Key**

```bash
cat /home/user/legacy-wallet.dump|grep -E 'hdseed|master'
# extended private masterkey: xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf
L1EasEYJHcHFUEHNT8D2W8yW2s9DzbhYjjCX8r8qrNa9Go7syAgg 2022-10-31T10:48:56Z hdseed=1 # addr=bc1ql84005zpv894v03spvenpjc6zdclxsj9e42qrv
```

> `L1EasEYJHcHFUEHNT8D2W8yW2s9DzbhYjjCX8r8qrNa9Go7syAgg` is the **256-bits hd seed** in WIF **compressed** format **from where** will always be generated the same **extended private masterkey**, if you exactly follow the BIP32 process of generating master keys.

- **Now create a legacy, but empty wallet**:

```bash
./bitcoin-cli -named createwallet wallet_name="legacy-wallet-restore" blank=true passphrase="my-passphrase" descriptors=false load_on_startup=false
```
```json
{
  "name": "legacy-wallet",
  "warning": "Wallet created successfully. The legacy wallet type is being deprecated and support for creating and opening legacy wallets will be removed in the future."
}
```

- **Unlock the wallet for a while**:

```bash
./bitcoin-cli -rpcwallet="legacy-wallet-restore" walletpassphrase "my-passphrase" 60
```

- **Set its hdseed with the one written down previously**:

```bash
./bitcoin-cli -named -rpcwallet="legacy-wallet-restore" sethdseed newkeypool=true seed=L1EasEYJHcHFUEHNT8D2W8yW2s9DzbhYjjCX8r8qrNa9Go7syAgg
```

- **Dump the wallet**:

```bash
./bitcoin-cli -rpcwallet=legacy-wallet-restore dumpwallet "legacy-wallet-restore.dump"
{
  "filename": "/home/user/legacy-wallet-restore.dump"
}
```

- **Retrieve secrets, and check they are the same**:

```bash
cat /home/user/legacy-wallet-restore.dump|grep -E 'hdseed|master'
extended private masterkey: xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf
L1EasEYJHcHFUEHNT8D2W8yW2s9DzbhYjjCX8r8qrNa9Go7syAgg 2022-10-31T11:00:53Z hdseed=1 # addr=bc1ql84005zpv894v03spvenpjc6zdclxsj9e42qrv
```

> The wallet has been restored from a written-down **hdseed**.

- **Backup the wallet**:

```bash
./bitcoin-cli -named -rpcwallet="legacy-wallet" backupwallet destination="legacy-wallet.back"
```

> Using a full wallet backup remains the sure way to save everything that could have been generated since the wallet creation, such as previous `hdseed`, addresses labels... Always have an up-to-date wallet file backup.

- **Migrate the wallet**:

```bash
./bitcoin-cli -rpcwallet="legacy-wallet" getwalletinfo
{
  "walletname": "legacy-wallet",
  "walletversion": 169900,
  "format": "bdb", # notice this
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoololdest": 1667213337,
  "keypoolsize": 1000, # notice this
  "hdseedid": "4542f371131acb30330b303e56cb6141d0f7eaf9",
  "keypoolsize_hd_internal": 1000,
  "unlocked_until": 1667215403,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": false, # notice this
  "external_signer": false
}

./bitcoin-cli -rpcwallet="legacy-wallet" walletpassphrase "my-passphrase" 600

./bitcoin-cli -rpcwallet=legacy-wallet migratewallet
# <A few minutes are required>
{
  "wallet_name": "legacy-wallet",
  "backup_path": "/home/user/.bitcoin/wallets/legacy-wallet/legacy-wallet-1667216212.legacy.bak"
}

./bitcoin-cli -rpcwallet="legacy-wallet" getwalletinfo
{
  "walletname": "legacy-wallet",
  "walletversion": 169900,
  "format": "sqlite", # notice this
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoolsize": 4000, # notice this
  "keypoolsize_hd_internal": 4000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true, # notice this
  "external_signer": false
}
```

- **Dump the private descriptors of the migrated wallet**:

```bash
./bitcoin-cli -rpcwallet="legacy-wallet" listdescriptors true
```

```json
{
  "wallet_name": "legacy-wallet",
  "descriptors": [
    {
      "desc": "combo([f9eaf7d0]L1EasEYJHcHFUEHNT8D2W8yW2s9DzbhYjjCX8r8qrNa9Go7syAgg)#dg99hqe3",
      "timestamp": 1667216196,
      "active": false
    },
    {
      "desc": "combo(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/0'/0'/*')#4pmzfqnz",
      "timestamp": 0,
      "active": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "combo(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/0'/1'/*')#983nesxm",
      "timestamp": 0,
      "active": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "pkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/44'/0'/0'/0/*)#9qf3g0e3",
      "timestamp": 1667216300,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "pkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/44'/0'/0'/1/*)#55vs46ff",
      "timestamp": 1667216300,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/49'/0'/0'/0/*))#yu27rhtc",
      "timestamp": 1667216300,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/49'/0'/0'/1/*))#zlzmc6qv",
      "timestamp": 1667216300,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/86'/0'/0'/0/*)#k2jg2qgg",
      "timestamp": 1667216300,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/86'/0'/0'/1/*)#87hfh4cs",
      "timestamp": 1667216300,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/84'/0'/0'/0/*)#lfh4zm2d",
      "timestamp": 1667216300,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh(xprv9s21ZrQH143K4AbeRBAeN9wbP21AMnTpxBmBfJ6464DyJEEbrpdiibzPhuB5rJRyf4EVVumEzu9E2u8stWfY4wbfJbT15PsdJJb7yapphHf/84'/0'/0'/1/*)#waj5lw64",
      "timestamp": 1667216300,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    }
  ]
}
```

In the migrated wallet, the original `hdseed` is still present as well as the expected **Extended Master Private Key**.

There are also **2 descriptors per address type** in the wallet, one for payment, one for change (identified with `internal: true` and the `/1` part).

We also notice that an active `taproot` descriptor has automatically been added for supporting taproot addresses in the wallet. Taproot is unsupported in legacy wallets.

# Restoring a wallet from scratch

> From a Standalone offline version of https://github.com/iancoleman/bip39

- **BIP39 then BIP32 process reminder**:
```
From 256 bits of entropy => Encoded in a 24 words mnemonic => PBKDF2 of mnemonic and a passphrase (default "") using BIP39 default settings (2048 iterations) => Yields a 512 bits "BIP39 Seed".

"BIP39 Seed" => BTC coin: HMAC-SHA512 with "Bitcoin seed" as the key and the Seed as data with default BIP32 settings => Yield a "BIP32 root key".
```

**Sample BIP32 root key**:

> xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p

**This is the key that will be set for all the descriptor of our wallet.**

Indeed, from our seed, to recreate the (modern) wallet, we need to **generate the descriptors** to have in our wallet with two descriptor for each of the [HD wallet standard](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md#features) you want supported by your wallet, such as:

- Pay-to-pubkey-hash scripts (P2PKH), through the `pkh` function (purpose id **[44'](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki#purpose)**).

- Pay-to-witness-pubkey-hash scripts (P2WPKH), through the `wpkh` function (purpose id **[84'](https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki#public-key-derivation)**).

- P2WPKH-nested-in-P2SH based accounts, through the `sh(wpkh())` nested functions (purpose id **[49'](https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki#public-key-derivation)**).

- Pay-to-taproot outputs (P2TR), through the `tr` function (purpose id **[86'](https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki#public-key-derivation)**).
- ...

[Reference](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md#reference)

We'll go only for P2PKH to keep things easy.


- **Generate the descriptors checksum**:

```bash
# P2PKH - pkh() function, derivation path "44'/0'/0'", payment ("/0"), unhardened address "/*":
./bitcoin-cli -named getdescriptorinfo descriptor="pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/0/*)"

{
  "descriptor": "pkh(xpub661MyMwAqRbcGB9EnCoeHgGNsN3utgzu8K4LzE7yMk5CnopnkMxP5gL4bqyWxA2gfqgU1SpMSDuckL1TyH5E5UFPghkMjY19nHg9UbJx4y6/44'/0'/0'/0/*)#zjhd60an",
  "checksum": "axglv7h8", # notice this
  "isrange": true, # notice this
  "issolvable": true,
  "hasprivatekeys": true
}

# P2PKH - pkh() function, derivation path "44'/0'/0'", change ("/1"), unhardened address "/*":
./bitcoin-cli -named getdescriptorinfo descriptor="pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/1/*)"

{
  "descriptor": "pkh(xpub661MyMwAqRbcGB9EnCoeHgGNsN3utgzu8K4LzE7yMk5CnopnkMxP5gL4bqyWxA2gfqgU1SpMSDuckL1TyH5E5UFPghkMjY19nHg9UbJx4y6/44'/0'/0'/1/*)#nxjv86dt",
  "checksum": "vjd73t8l", # notice this
  "isrange": true, # notice this
  "issolvable": true,
  "hasprivatekeys": true
}

# Generate all the others descriptors you need for your wallet (wpkh, tr...)
```

- **Generate the complete JSON data of each of the descriptors to import in the wallet**:

This step is about creating the full JSON data structure required by the Bitcoin Client to import the descriptor info previously generated.

You will note that the previous commands are here to simply generate the required *checksum* to pass to the *importdescriptors* RPC call.

**JSON structures:**

Our *payment* descriptor must be **active** and not **[internal](https://bitcoincore.org/en/doc/22.0.0/rpc/wallet/importdescriptors/)**, with a **range**:

```json
[{ "desc": "pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/0/*)#axglv7h8", "timestamp": "now", "internal": false, "active": true, "range": [0, 999], "next_index": 0 }]
```

The *change* descriptor must be **active** and **internal**, with a **range**:

```json
[{ "desc": "pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/1/*)#vjd73t8l", "timestamp": "now", "internal": true, "active": true, "range": [0, 999], "next_index": 0 }]
```

**Now escape the double quotes for bash processing:**

> Replace the " with \\" in the previous JSON strings:
```
# P2PKH, payment descriptor:
[{ \"desc\": \"pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/0/*)#axglv7h8\", \"timestamp\": \"now\", \"internal\": false, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]

# P2PKH, change descriptor:
[{ \"desc\": \"pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/1/*)#vjd73t8l\", \"timestamp\": \"now\", \"internal\": true, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]

```

> Beware of the [(unix) timestamp](https://bitcoincore.org/en/doc/22.0.0/rpc/wallet/importdescriptors/) set here: If we create a new wallet, a timestamp with "`now`" is ok, else you may need to use an earlier one in order to trigger a proper blockchain rescan after the import.

- **Create a descriptor, empty wallet**:

```bash
$ ./bitcoin-cli -named createwallet wallet_name="wallet-desc" blank=true passphrase="my-passphrase" descriptors=true load_on_startup=false

# Unlock the wallet
$ ./bitcoin-cli -rpcwallet="wallet-desc" walletpassphrase "my-passphrase" 120

# Wallet info
$ ./bitcoin-cli -rpcwallet="wallet-desc" getwalletinfo
{
  "walletname": "wallet-desc",
  "walletversion": 169900,
  "format": "sqlite",
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoolsize": 0,
  "keypoolsize_hd_internal": 0,
  "unlocked_until": 1667230210,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true,
  "external_signer": false
}
```

- **Import all the generated descriptors into the wallet**:

```bash
# P2PKH, payment descriptor:
$ ./bitcoin-cli -rpcwallet="wallet-desc" importdescriptors "[{ \"desc\": \"pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/0/*)#axglv7h8\", \"timestamp\": \"now\", \"internal\": false, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]"
[
  {
    "success": true
  }
]

# P2PKH, change descriptor:
$ ./bitcoin-cli -rpcwallet="wallet-desc" importdescriptors "[{ \"desc\": \"pkh(xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p/44'/0'/0'/1/*)#vjd73t8l\", \"timestamp\": \"now\", \"internal\": true, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]"
[
  {
    "success": true
  }
]

# Add all the others descriptors you need...

$ ./bitcoin-cli -rpcwallet="wallet-desc" getwalletinfo
{
  "walletname": "wallet-desc",
  "walletversion": 169900,
  "format": "sqlite",
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoolsize": 1000,
  "keypoolsize_hd_internal": 1000,
  "unlocked_until": 1667234490,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true,
  "external_signer": false
}
```

**Notes (cheat sheet):**

As of `Bitcoin Core 0.24-rc1`, **newly created descriptor wallets** have the **following private descriptors** created by default:

```bash
./bitcoin-cli -named createwallet wallet_name="wallet-desc-full" blank=false descriptors=true load_on_startup=false

./bitcoin-cli -rpcwallet="wallet-desc-full" listdescriptors true
```

```json
{
  "wallet_name": "wallet-desc-full",
  "descriptors": [
    {
      "desc": "pkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/44'/0'/0'/0/*)#95k6c8ht",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/49'/0'/0'/0/*))#chqw6jt8",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/84'/0'/0'/0/*)#er45lhmv",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/86'/0'/0'/0/*)#u0w8d03d",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "pkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/44'/0'/0'/1/*)#5qnm9j8n",
      "timestamp": 1667763996,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/49'/0'/0'/1/*))#75gtplqn",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/84'/0'/0'/1/*)#ghs4zzt5",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr(xprv9s21ZrQH143K4TeH31hLaQhNMHXPHjn4uzgG6nQttQ7ZntzbRYkWFhc7GnnCpvTZZFH8SnvWrh76Bp78NzLEaHy7Jo5cgEV25zcdEhucawZ/86'/0'/0'/1/*)#dmtxs6p4",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    }
  ]
}
```

- And their matching **Public descriptors** (for *watch-only* wallets):

```bash
./bitcoin-cli -rpcwallet="wallet-desc-full" listdescriptors

{
  "wallet_name": "wallet-desc-full",
  "descriptors": [
    {
      "desc": "pkh([9c1d8530/44'/0'/0']xpub6BxaU8rZM56qjb3pijsnS83ubUVyMeDGCZn7XX9yS112UXPFZsPFYnxnP6XCgsCBZat97cWLgcnysvKBkgoXw9kUa8k7u8hzvBmKEmuu6kM/0/*)#59kyljz0",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh([9c1d8530/49'/0'/0']xpub6CS9MFAPJaANjDr9NMMimjz1kq4mWNaAfTecrUUph7k6vp9GdophqeXPoVdcBUTWhmmGPZfh9pJmoBaTxpsRfM2eazTem8QcdeQ1m6MHge5/0/*))#sv67a5z3",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh([9c1d8530/84'/0'/0']xpub6DBWPwWPcMhUfc5d4NR7amGSWDCLBHHG6bCT8yNijdiN1HA9rQGamkkUiT7BPxdAWHmjjbToL9RsgMLTxxTmW7cn4zxTbrAo4NtrHZKJ6gk/0/*)#ctr5tzqp",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr([9c1d8530/86'/0'/0']xpub6Bv6HJSBu7sGeQPH619CgNCD38oc1T14TPrL8dCZEHeamCpddc4Fj1FEswGnDtUNU95wbHinFRwndoq1JK83bipEnwZ4V3omX2dBjjA5RTJ/0/*)#9ws08lps",
      "timestamp": 1667763996,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "pkh([9c1d8530/44'/0'/0']xpub6BxaU8rZM56qjb3pijsnS83ubUVyMeDGCZn7XX9yS112UXPFZsPFYnxnP6XCgsCBZat97cWLgcnysvKBkgoXw9kUa8k7u8hzvBmKEmuu6kM/1/*)#93n9z8jh",
      "timestamp": 1667763996,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "sh(wpkh([9c1d8530/49'/0'/0']xpub6CS9MFAPJaANjDr9NMMimjz1kq4mWNaAfTecrUUph7k6vp9GdophqeXPoVdcBUTWhmmGPZfh9pJmoBaTxpsRfM2eazTem8QcdeQ1m6MHge5/1/*))#9d5g9thw",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "wpkh([9c1d8530/84'/0'/0']xpub6DBWPwWPcMhUfc5d4NR7amGSWDCLBHHG6bCT8yNijdiN1HA9rQGamkkUiT7BPxdAWHmjjbToL9RsgMLTxxTmW7cn4zxTbrAo4NtrHZKJ6gk/1/*)#flx4khse",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "tr([9c1d8530/86'/0'/0']xpub6Bv6HJSBu7sGeQPH619CgNCD38oc1T14TPrL8dCZEHeamCpddc4Fj1FEswGnDtUNU95wbHinFRwndoq1JK83bipEnwZ4V3omX2dBjjA5RTJ/1/*)#564w623g",
      "timestamp": 1667763997,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    }
  ]
}

```

You will notice that:

- No descriptor refers to an `hdseed` anymore in descriptor wallets.

- **Unhardened addresses** is now [used by default](https://bitcoin.stackexchange.com/questions/113834/mainnet-wallet-will-refuse-to-dump-privkeys#comment129650_113838) with descriptor wallets, as specified with the "`/*`" final path (rather than "`/*'`"). Derivation path must use hardened derivation steps (`'`).

## Misc

- **How to generate descriptors with scripting?**

> https://github.com/brianddk/reddit/blob/010934f/python/hdseed.py

- **How can I quickly know that the address I just generated is still from the seed I have written down, and so my wallet hasn't changed without me knowing?**

**For any wallets:**

```bash
# Note: We only have imported descriptors for "pkh" addresses in this wallet
$ ./bitcoin-cli -rpcwallet="wallet-desc" getnewaddress "" legacy
14XcVrUK68zzwe6ekA6JacvgYDkm5fj8ks

$ ./bitcoin-cli -rpcwallet="wallet-desc" getaddressinfo 14XcVrUK68zzwe6ekA6JacvgYDkm5fj8ks | grep -E 'ismine|ischange|solvable|hdmasterfingerprint'
  "ismine": true,
  "solvable": true,
  "ischange": false,
  "hdmasterfingerprint": "cf302f59", # note this
```

**The `hdmasterfingerprint` of the address must match the first 4 bytes of the `RIPEMD-160` hash of the wallet's "Extended master Public Key". It is the private key (`xprv...`) set for the descriptor that generated the address.**

The process is:
- Compute the corresponding [Extended Master Public Key](https://learnmeabitcoin.com/technical/extended-keys)
- Deserialize it to get the `public key` bits
- hash160 it

To implement it, follow the [BIP32 specs](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki#extended-keys) or go with offline scripting with the following code and 3rd party lib:

```python
#!/usr/bin/env python3
from electrum.bip32 import BIP32Node
from sys import argv

hdmasterfingerprint = BIP32Node.from_xkey(argv[1]).calc_fingerprint_of_this_node().hex().lower()
print(hdmasterfingerprint)
```

```bash
./hdmasterfingerprint_from_xprv.py xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p
# cf302f59
```

**- How to create a watch-only descriptor wallet?**

- Export the *public descriptors* of your wallet:

```bash
./bitcoin-cli -rpcwallet="wallet-desc" listdescriptors
{
  "wallet_name": "wallet-desc",
  "descriptors": [
    {
      "desc": "pkh([cf302f59/44'/0'/0']xpub6CHT4rUQYwXJqgxRfoyd25K5yQBdXiL58K2c9oceVg5A3uMNNgpMJNq73MSgnTzeNgdVdwYBDcGQR3bUdumNQzH1263SPH7nw83NUZQyXyw/0/*)#c5yuzacn",
      "timestamp": 1231006505,
      "active": true,
      "internal": false,
      "range": [
        0,
        999
      ],
      "next": 0
    },
    {
      "desc": "pkh([cf302f59/44'/0'/0']xpub6CHT4rUQYwXJqgxRfoyd25K5yQBdXiL58K2c9oceVg5A3uMNNgpMJNq73MSgnTzeNgdVdwYBDcGQR3bUdumNQzH1263SPH7nw83NUZQyXyw/1/*)#fqpalggt",
      "timestamp": 1231006505,
      "active": true,
      "internal": true,
      "range": [
        0,
        999
      ],
      "next": 0
    }
  ]
}
```

- Create an *empty wallet* with *no private keys support*:

```bash
./bitcoin-cli -named createwallet wallet_name="wallet-watch" disable_private_keys=true blank=true descriptors=true load_on_startup=false
```

- Import the *public descriptors*:

```bash
./bitcoin-cli -rpcwallet="wallet-watch" importdescriptors "[{ \"desc\": \"pkh([cf302f59/44'/0'/0']xpub6CHT4rUQYwXJqgxRfoyd25K5yQBdXiL58K2c9oceVg5A3uMNNgpMJNq73MSgnTzeNgdVdwYBDcGQR3bUdumNQzH1263SPH7nw83NUZQyXyw/0/*)#c5yuzacn\", \"timestamp\": \"now\", \"internal\": false, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]"

./bitcoin-cli -rpcwallet="wallet-watch" importdescriptors "[{ \"desc\": \"pkh([cf302f59/44'/0'/0']xpub6CHT4rUQYwXJqgxRfoyd25K5yQBdXiL58K2c9oceVg5A3uMNNgpMJNq73MSgnTzeNgdVdwYBDcGQR3bUdumNQzH1263SPH7nw83NUZQyXyw/1/*)#fqpalggt\", \"timestamp\": \"now\", \"internal\": true, \"active\": true, \"range\": [0, 999], \"next_index\": 0 }]"
```

**- Converting xpub keys**

Public descriptors of default wallets created with Bitcoin Core use master extended public keys starting with `xpub`. For instance our default descriptor wallet created above has the following key for the P2WPKH standard (BIP84):

```
xpub6DBWPwWPcMhUfc5d4NR7amGSWDCLBHHG6bCT8yNijdiN1HA9rQGamkkUiT7BPxdAWHmjjbToL9RsgMLTxxTmW7cn4zxTbrAo4NtrHZKJ6gk
```

If you import this key in some mobile wallet for a watch-only purpose, you may obtain the [BIP44 addresses](https://github.com/BlueWallet/BlueWallet/issues/2958) rather than the expected *BIP84* addresses. A solution is to convert the key to its `zpub` [equivalent](https://github.com/satoshilabs/slips/blob/master/slip-0132.md#registered-hd-version-bytes) with a tool such as https://github.com/jlopp/xpub-converter.


**- What about Ethereum?**

As long as you have a `BIP32 Root Key`, you can use it for your Ethereum wallets with any clients such as `geth`, `metamask`...

> The standard **BIP32 Derivation Path** for Ethereum is `m/44'/60'/0'/0`.

For instance, using an offline version of https://iancoleman.io/bip39/:

- **Given the BIP32 root key**:

```
xprv9s21ZrQH143K3h4mgBGdvYKeKLDRVEH3m68kBqiMoQYDv1VeCpe8Xt1akb1fNoh7hZcHAhfwggeeyFtdjFJRLyJGGAp7Te1mgZh5dHL9B6p
```

- **Coin**: Ethereum

- **Derivation path**: BIP44

- The first (non-hardened) derived address can be used as your first account:

    - **Private key**: `0xa45ed09c1c0a70533078f980cc8ee58756ddbef012dcfe6cbece7cb49ab258eb`
    - **Account (address)**:
    `0x6502F075F2ec9492C9Dd64E989798e5BFF51E4AE`
