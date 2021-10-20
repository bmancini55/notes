# BIP32 - HD Keys for classic transactions

https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki

This BIP:

- Defines xpub/xprv encodings which are HD extended keys for legacy addresses on mainnet
- Defines child key derivation functions for:
  - private key => private key
  - private key => public key
  - public key => public key (non-hardened addresses only)
- Can define up to 255 accounts/depths
- Can define 2<sup>31</sup> normal child keys and 2<sup>31</sup> hardened chlid keys
- Normal keys are indexed 0 through 2<sup>31</sup>-1
- Hardened keys are indexed 2<sup>31</sup> through 2<sup>32</sup>-1
- Derivation uses HMAC-SHA512.
- The general idea of derivation is that we use the `chain_code` as the
  key, then either the public key + index or private key + index as the
  data.
- The result 64-bytes are split in 2. The first half is `tweakAdd` by
  the existing key and becomes the child key. The second half is the child
  `chain_code`.

### Hierarchy Recommendations

- depth 0 - master key
- depth 1 - accounts: m/0, m/1, m/i
- depth 2 - chains : m/0/0
- depth 3 - addresses

### Normal vs Hardened:

- Non-hardened keys are useful because you can derive child public keys given the parent public key, you do not need the private key.
- This feature distinguishes normal from hardened keys (which public keys cannot be used to derive child public keys)
- Knowledge of a parent extended pubkey plus a normal private key descending from it is equivalent to knowing the parent's extented private key (and thus every private key) descending from it.
- This last bit is the reason for hardened keys
- Hardened keys are recommened for the account level in the tree
- This prevents a leak of account-specific private keys from compromising the master key

### Encoding/Decoding

Base58Check is used to define the

```
4 bytes: version bytes
1 bytes: depth, starting at 0x00
4 bytes: fingerprint of the parent's key (0x00000000 for master key)
4 bytes: child number
32 bytes: chain code
33 bytes: public key or private key (0x00 prefix)
```

### Master Key Generation

- Generate a seed `S` (recommend 256bits)
- Calculate `l = HMAC-SHA512(Key = "Bitcoin seed", Data = S)`
- Split `l` into two 32-byte sequences `l_L` and `l_R`
- Master secret i= `l_L`
- Master chain code = `l_R`

If `l_L` is 0 or >= n, master key is invalid and you should generate
a new seed.

### Private Key => Child Private Key

Mathematically the derivation is defined as:

```
CKDPriv((k_par, c_par), i) => (k_i, c_i)
```

This method can derive both hardened and non-hardened child private keys.

Derivation for hardened keys uses the private key as part of the data. We
prepend a 0x00 byte to the front to maintain the same 33-bytes of data
as used by non-hardened keys.

```
l_hard = HMAC-SHA512(Key = c_parent, Data = 0x00 || k_parent || i)

```

where:
`c_parent` is 32-bytes `chain_code`
`k_parent` is the 32-byte secp256k1 private key
`i` is the uint32be encoding of the child number

For non-hardened keys the formula is:

```
l_norm = HMAC-SHA512(Key = c_parent, Data = K_parent || i)
```

where:

`c_parent` is 32-bytes
`K_parent` is the 33-byte compressed SEC encoding of a valid secp256k1 point
`i` is the uint32be encoding of the child number

This same derivation is used in pub => pub derivation for non-hardened keys.

The HMAC-SHA512 results in a 64-byte result that is split into two
32-byte sequences: `l_L` and `l_R`.

The child private key is derived via `tweakAdd` of the private key value

```
k_i = l_L + k_parent
c_i = l_R
```

### Private Key => Child Public Key

So we can always convert a private key into a public key using standard
secp256k1 scalar multiplication against the generator point `G`.

So...

```
K_i = k_i * G
```

As a result, we should always just derive the child private key first
then convert it to the public key.

Otherwise, if we convert to a public key first, we can only do pubkey
derivation for non-hardened keys!

### Public Key => Child Private Key Derivation

Yeah... not possible

### Public Key => Child Public Key Derivation

Mathematically the derivation is defined as:

```
CKDpub((K_parent, c_parent), i) -> (K_i, c_i)
```

This method fails if there is an attempt to derive a hardened
child key since hardened keys use the private key as the data
in the HMAC.

Otherwise we use the same derivation formula as we do with non-hardened
private keys!

```
l = HMAC-SHA512(Key = c_parent, Data = K_parent || i)
```

where:

`c_parent` is 32-bytes
`K_parent` is the 33-byte compressed SEC encoding of a valid secp256k1 point
`i` is the uint32be encoding of the child number

The 64-byte result is split into two 32-byte sequences: `l_L` and `l_R`.

The child public key is thus:

```
K_i = l_L * G + K_parent
c_i = l_R
```

Meaning we use point addition of the parent's public key and take
`l_L * G` to generate the point for the child's point.

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

# Segwit Wallet Development

https://bitcoincore.org/en/segwit_wallet_dev/

```

```
