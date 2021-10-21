# BIP173 - Bech32 Address Format for v0 witness addresses

https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki
https://medium.com/@MeshCollider/some-of-the-math-behind-bech32-addresses-cf03c7496285

### Bech32

- base32 encoding
- uses 5-bit words
- fixed case is easier to read/transcribe
- bech contains characters from BCH (error detection algorithm used) and sounds similar to "base"
-
- implements a BCH code that guarantees detection of any error affecting at most 4 characters
- can be either uppercase or lowercase, but not both
- human-readable part - intended to convey the type of data or anything else
  that is relevant to the reader - must be 1 to 83 US-ASCII characters [33-126] - allows character "1"
- separator, always "1"
  - last "1" in HRP
- data part

  - at least 6 characters long and only consists of alphanumeric characters
  - excludes "1", "b", "i", and "o"

- checksum

  - last 6 characters of the checksum
  - covers the hrp by passing the upper bits, followed by a 0, followed by the lower bits > this is necessary because bech32 uses 5-bit words

- encoders must always process as lowercase

### SegWit address format

- addresses are always between 14 and 74 characters long
- addresses start with "bc1" or "tb1" for mainnet and testnet
- must be careful converting witness version into proper opcode, OP_1 = 0x51 not 0x01.

#### Encoding

- HRP of "bc" for mainnet
- HRP of "tb" for testnet
- data part
  - 1 character (5 bits) with the witness version
  - Conversion of the 2-40-byte witness program (v0 will be only 20-byte or 32-byte)
    - start with MSB
    - rearrange bits into groups of 5, pad with zeroes at the end if needed
    - translate using character table above

#### Decoding

- verify HRP is valid
- verify first word of data is 0 to 16 (valid witness version)
- data part
  - start with MSB
  - rearrange each 5-bit word into an 8-bit word
  - any excess bits must be 4 or less, MUST be all zeroes, and can be discarded
  - there should be 2 to 40 groups which are bytes

# BIP350 - Bech32m format for v1+ witness addresses

https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki

- improved variant of Bech32 called Bech32m.
- ammends BIP173 to use Bech32m for native seg wit outputs version 1 to 16
- Bech32 remains in use for segregated witness output version 0
- Bech32 has a weakness
  - the final character is a `p`, inserting or deleting any number of `p`
    characters immediately preceding it does not invalidate the checksum.
- Weakness does not effect existing uses of witness version 0 due to the
  restrictions on lengths (20-byte or 32-byte)
- difference is the use of a new constant that is xor'd into the checksum
  at the end
- all other aspects remain unchanged
