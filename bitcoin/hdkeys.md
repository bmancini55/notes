# BIP32 - HD Keys for classic transactions

https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki

- Defines xpub/xprv which are mainnet prefixes

- Defines child key derivation functions for:

  - private key -> private key
  - private key -> public key
  - public key -> public key

- Public key -> public key derivation only works for non-hardened addresses

- Can define up to 255 accounts
- Can define 2<sup>31</sup> normal child keys and 2<sup>31</sup> hardened chlid keys
- Normal keys are indexed 0 through 2<sup>31</sup>-1
- Hardened keys are indexed 2<sup>31</sup> through 2<sup>32</sup>-1

### Normal vs Hardened:

- Non-hardened keys are useful because you can derive child public keys given the parent public key, you do not need the private key.
- This feature distinguishes normal from hardened keys (which public keys cannot be used to derive child public keys)
- Knowledge of a parent extended pubkey plus a normal private key descending from it is equivalent to knowing the parent's extented private key (and thus every private key) descending from it.
- This last bit is the reason for hardened keys
- Hardened keys are recommened for the account level in the tree
- This prevents a leak of account-specific private keys from compromising the master key

### Serialization

Base58Check is used to define the

```
4 bytes: version bytes
1 bytes: depth, starting at 0x00
4 bytes: fingerprint of the parent's key (0x00000000 for master key)
4 bytes: child number
32 bytes: chain code
32 bytes: public key or private key (0x00 prefix)
```

# BIP49 - Derivation scheme for P2SH-P2WPKH accounts

https://github.com/bitcoin/bips/blob/master/bip-0049.mediawiki

- Defines ypub
- Defined for P2SH-P2WPKH (BIP141) transactions
- Defines seperate segwit accounts to ensure that wallets compatible with
  this BIP will detect the accounts and handle them appropriately
- Follows BIP44
- Changes `purpose` path-level to `49'`

Path becomes:

```
m / 49' / coin_type' / account' / change / address_index
```

# BIP84 - Derivation scheme for P2WPKH

https://github.com/bitcoin/bips/blob/master/bip-0084.mediawiki

- Defines zpub
- Defines the derivation schems for HD wallets using P2WPKH (BIP173)
  serialization format for segwit transactions
- Allows the user to use different HD wallets with the same masterseed
- User creates dedicated segregated witness accounts, which ensures that
  only wallets compatible with this BIP will detect accounts and handle them
- Folows BIP44
- Changes `purpose` path-level to `84'`

# BIP43 - Purpose Field for Deterministic Wallets

https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki

# BIP44 - Multi-Account Hierarchy for Deterministic Wallets

https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki

- defines a logical hierarchy for deterministic wallets based on BIP32
- this BIP is a particular application of BIP43
- purpose of the hierarchy allows handling of:
  - multiple coins
  - multiple accounts
  - external and internal chains per account
  - millions of addresses per chain
- defines five levels

Defined levels where apostrophe indicates hardened derivation path

```
m / purpose' / coin_type' / account' / change / address_index
```

# BIP39 - Mnemonic code for deterministic key generation

https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
