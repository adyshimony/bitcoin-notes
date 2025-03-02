# Bitcoin Wallet Descriptors Tutorial


## Introduction

Wallet descriptors (sometimes called "output descriptors") are a formalized language for describing collections of Bitcoin output scripts (scriptPubKeys) along with information needed to spend from them. They were introduced to Bitcoin Core in version 0.17 and have been continuously improved since, with subsequent versions adding support for more advanced features like Miniscript (v24), Miniscript signing support (v25), and Miniscript in Taproot descriptors (v26).

Before descriptors, wallets relied on implicit assumptions that were often unclear and led to compatibility issues between different wallet implementations. Descriptors solve this by providing:

1. A clear, unambiguous specification of which addresses belong to a wallet
2. All information needed to detect and spend from these addresses
3. A standardized format that different wallets can understand

### Why do we need descriptors?

Descriptors serve several important practical purposes in the Bitcoin ecosystem:

1. **Wallet Import/Export**: Descriptors can be imported into compatible wallets (like Bitcoin Core) to recreate the exact same wallet structure across different devices or applications.

2. **Watch-only Wallets**: You can import a descriptor containing only public keys to create a watch-only wallet that can monitor balances and transactions without the ability to spend funds.

3. **Hardware Wallet Integration**: Descriptors allow software wallets to communicate precisely with hardware wallets about which addresses to generate and monitor.

4. **Multi-wallet Coordination**: In multi-signature setups, descriptors ensure all participants are watching the same set of addresses.

5. **Wallet Recovery**: By storing your wallet descriptor securely, you can recover your entire wallet structure even if your original wallet software or device is lost.

6. **Blockchain Scanning**: Descriptors can be used with commands like `scantxoutset` to quickly find funds associated with your keys without importing them to your wallet.

7. **Automated Systems**: Services that generate addresses for users (like exchanges or payment processors) can use descriptors to systematically manage large numbers of addresses.

## Descriptor Fundamentals

### Basic Descriptor Syntax

A descriptor consists of a function-like expression:

```
FUNCTION(KEY)#CHECKSUM
```

Where:
- `FUNCTION` defines the output script type (e.g., pkh for P2PKH, wpkh for P2WPKH)
- `KEY` specifies the key information
- `CHECKSUM` is an optional 8-character checksum to detect errors

#### Example Descriptors:

For a legacy P2PKH address:
```
pkh(02c6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5)
```

For a native SegWit P2WPKH:
```
wpkh(02f9308a019258c31049344f85f89d5229b531c845836f99b08601f113bce036f9)
```

### Key Expressions in Descriptors

Keys can be expressed in several formats:

1. **Raw Public Keys**: Hex-encoded public keys
   ```
   pkh(02e7d75b449e7a7e6511a1af4e79acaed055c1ee16602b9f4787a15b74d0ae5122)
   ```

2. **Extended Keys (xpub/xprv)**: For HD wallets
   ```
   wpkh(xpub6D...Np6/0/*)
   ```
   Here, `/0/*` indicates derivation path (change level 0, all addresses)

3. **Key Origin Information**: For improved accountability
   ```
   wpkh([d34db33f/84h/0h/0h]xpub.../0/*)
   ```
   The `[...]` contains the master key fingerprint and derivation path

4. **WIF Keys**: For private keys in WIF format
   ```
   pkh(5KYZdUEo39z3FPrtuX2QbbwGnNP5zTd7yyr2SC1j299sBCnWjss)
   ```

### Common Descriptor Functions

1. **pk()** - Pay to Public Key (P2PK)
2. **pkh()** - Pay to Public Key Hash (P2PKH, legacy addresses)
3. **wpkh()** - Pay to Witness Public Key Hash (P2WPKH, native SegWit)
4. **sh(wpkh())** - Pay to Script Hash wrapping a Witness Public Key Hash (nested SegWit)
5. **multi()** - Bare multisig (rarely used directly)
6. **sortedmulti()** - Multisig with keys sorted lexicographically
7. **sh(multi())** - P2SH multisig
8. **wsh(multi())** - P2WSH multisig
9. **sh(wsh(multi()))** - Nested SegWit multisig
10. **tr()** - Pay to Taproot (P2TR)
11. **multi_a()** - Multisig for Taproot script paths using OP_CHECKSIG and OP_CHECKSIGADD
12. **sortedmulti_a()** - Like multi_a but with keys sorted lexicographically
13. **addr()** - Any supported address type
14. **raw()** - Raw hex scripts

### Real-world Applications of Descriptors

When you see a descriptor like `pkh(02c6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5)`, this represents a single legacy P2PKH address derived from that specific public key. In real-world applications, this is used in several contexts:

1. **Watch-only Wallets**: When you want to monitor an address without having the private key
2. **Hardware Wallet Integration**: Importing public keys from hardware wallets for monitoring
3. **Exchange Deposits**: Exchanges may use single-key descriptors for user deposit addresses
4. **Simple Wallet Recovery**: Recovering funds when you have the public key but need to specify the exact script type

### Descriptor Implementation in Bitcoin Core

Bitcoin Core implements descriptors through the `descriptor.cpp` and related files. The parser and evaluation logic allow Bitcoin Core to:

1. Parse descriptors from string format
2. Generate addresses from descriptors
3. Create scriptPubKeys for transactions
4. Monitor for incoming transactions matching descriptors
5. Sign transactions using the appropriate script formats

## Derivation Paths in Depth

### BIP32 Derivation Path Structure

Derivation paths follow BIP32 specification and typically look like:

```
m / purpose' / coin_type' / account' / change / address_index
```

Where:
- `m` represents the master key
- `purpose'` is usually a BIP number (44', 49', 84')
- `coin_type'` is 0' for Bitcoin mainnet, 1' for testnet
- `account'` is the account number, starting at 0'
- `change` is 0 for external addresses, 1 for internal (change)
- `address_index` is the address number in sequence

The apostrophe (') indicates hardened derivation, which provides better security but prevents deriving child public keys from parent public keys.

### Common Derivation Path Examples

Derivation paths play a crucial role in HD (Hierarchical Deterministic) wallets, allowing a single seed to generate countless addresses. Each BIP specifies a standard path format for different address types.

#### Understanding the Path Structure

Let's break down what each part of a derivation path means, using BIP44 as an example:

`m/44'/0'/0'/0/*`

- `m` - Master key (root of the HD tree)
- `44'` - Purpose (44' for BIP44 standard)
- `0'` - Coin type (0' for Bitcoin mainnet, 1' for testnet)
- `0'` - Account index (allows multiple accounts from same seed)
- `0` - Change (0 for external/receiving addresses, 1 for internal/change addresses)
- `*` - Address index (incremental index for each new address)

The apostrophe (') indicates hardened derivation, which adds security by preventing parent public key from deriving child private keys.

#### Common BIP Standards and Their Descriptors

1. **BIP44 (Legacy P2PKH)**: `m/44'/0'/0'/0/*`

   ```
   pkh([fingerprint/44h/0h/0h]xpub.../0/*)
   ```
   
  This descriptor creates standard legacy Bitcoin addresses (starting with '1'). The `pkh()` function generates Pay-to-Public-Key-Hash scripts using keys derived from the xpub at path index 0/* (receiving addresses). When importing this descriptor, you're instructing the wallet to track all P2PKH addresses derived from this specific xpub following the BIP44 path convention.
   
   **Example address**: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa

2. **BIP49 (Nested SegWit)**: `m/49'/0'/0'/0/*`

   ```
   sh(wpkh([fingerprint/49h/0h/0h]xpub.../0/*))
   ```
   
  This creates P2SH-wrapped SegWit addresses (starting with '3'). The nested functions `sh(wpkh())` mean "create a Pay-to-Witness-Public-Key-Hash script, then wrap that in a Pay-to-Script-Hash." This provides SegWit benefits while maintaining compatibility with older wallets. The xpub is derived following the BIP49 path convention.
   

   
   **Example address**: bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4

3. **BIP86 (Taproot)**: `m/86'/0'/0'/0/*`

   ```
   tr([fingerprint/86h/0h/0h]xpub.../0/*)
   ```
   
Creates Taproot addresses (starting with 'bc1p' on mainnet). 
   
   **Example address**: bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqzk5jj0

#### What the Key Origin Information Means

The part like `[fingerprint/44h/0h/0h]` before the xpub provides key origin information:

- `fingerprint` - First 8 characters of the Hash160 of the master pubkey
- `/44h/0h/0h` - Derivation path from the master key to the xpub that follows

This helps wallets (especially hardware wallets) identify the derivation path of the xpub and allows the wallet to consistently derive the correct child keys.

#### Practical Example of xpub Usage in Descriptors

When you see a complete descriptor like:

```
pkh([d34db33f/44h/0h/0h]xpub6ERApfZwUNrhLCkDtcHTcxd75RbzS1ed54G1LkBUHQVHQKqhMkhgbmJbZRkrgZw4koxb5JaHWkY4ALHY2grBGRjaDMzQLcgJvLJuZZvRcEL/0/*)#abcdef12
```

It tells a wallet:
1. Use the xpub starting with "xpub6ER..."
2. This xpub was derived from master key with fingerprint d34db33f using path 44h/0h/0h
3. Generate P2PKH addresses for all receiving addresses (path/0/*)
4. The checksum abcdef12 validates the descriptor's integrity

Importing this descriptor allows a wallet to:
- Generate and recognize any receiving address from this account
- Derive the correct scriptPubKey for each address
- Monitor the blockchain for transactions to these addresses
- Sign transactions (if the corresponding private key is available)

Note: The 'h' suffix can be used as an alternative to the apostrophe (') to denote hardened derivation.

### Wildcard Expansion in Descriptors

In descriptors, the `*` wildcard is a powerful mechanism that enables a single descriptor to represent multiple addresses or scripts. It can appear in several positions and contexts:

#### 1. Normal Chain Usage: `/0/*` - All addresses in the external chain

This is the most common usage, where the wildcard represents all possible address indexes in a chain.

**Example:**
```
wpkh([73c5da0a/84h/0h/0h]xpub6CcGh8BQPxr9zssX4eG8CiGzToU6Y9b3f2s2wNw65p9xtr8ySL6eYJVWKJDhzBDwzAkvRuZuPuJXbLzCYLhD9pK5cT44xdWhKjgkBxwYrgT/0/*)
```

**What this means:**
- The wallet will track all addresses derived from paths:
  - `.../0/0`
  - `.../0/1`
  - `.../0/2`
  - And so on...
- When imported with `importdescriptors`, the wallet will scan for transactions to all these addresses.

#### 2. Range Specification: `/0/*` with range [0,100] - Specific address range

When importing a descriptor, you can specify exactly which indexes to use, limiting the scope of addresses.

**Example:**
```
bitcoin-cli importdescriptors '[{"desc": "wpkh([73c5da0a/84h/0h/0h]xpub6CcGh8BQPxr9zssX4eG8CiGzToU6Y9b3f2s2wNw65p9xtr8ySL6eYJVWKJDhzBDwzAkvRuZuPuJXbLzCYLhD9pK5cT44xdWhKjgkBxwYrgT/0/*)#rg53tfdn", "range": [0, 100], "timestamp": "now"}]'
```

**What this means:**
- Unlike the previous example, this explicitly limits scanning to indexes 0 through 100
- The wallet will only track addresses derived from:
  - `.../0/0`
  - `.../0/1`
  - ...
  - `.../0/100`
- Addresses with higher indexes (e.g., `.../0/101`) are ignored until the range is expanded

#### 3. Multiple Wildcards: `multi(1,xpub.../*/0,xpub.../*/0)` - Multiple HD chains

This is an advanced case where wildcards appear in non-standard positions, allowing representation of complex wallet structures.

**Example:**
```
wsh(multi(2,[73c5da0a/48h/0h/0h/1h]xpub6ERApfZwUNrhLCkDtcHTcxd75RbzS1ed54G1LkBUHQVHQKqhMkhgbmJbZRkrgZw4koxb5JaHWkY4ALHY2grBGRjaDMzQLcgJvLJuZZvRcEL/*/0,[f6a5d08f/48h/0h/0h/2h]xpub6EKgh9CbEN59UAXvXwKwYr7sRDDEzUYSZiF9KdV1cZVEYgmUxTZDrJDFACKcpxgHrMdRUJEcKUxiokZiWabVJhq8Q5KURDkZJRYeAz1Wq7P/*/0))
```

**What this means:**
- This creates multisig addresses where the signing keys come from different account paths
- The wildcard `*` is at the account level rather than the address index level
- When expanded with range [0,1], it generates addresses requiring 2 of 2 signatures from:
  - First account pair: Keys from `.../0/0` of both xpubs
  - Second account pair: Keys from `.../1/0` of both xpubs
- This allows creating multisig wallets where each signer uses different accounts for different purposes

#### 4. Tuple Notation for Receive/Change: `<0;1>` - Multiple descriptors from one

While not using the `*` wildcard, this related feature uses tuples to represent both receiving and change chains in a single descriptor.

**Example:**
```
wpkh([73c5da0a/84h/0h/0h]xpub6CcGh8BQPxr9zssX4eG8CiGzToU6Y9b3f2s2wNw65p9xtr8ySL6eYJVWKJDhzBDwzAkvRuZuPuJXbLzCYLhD9pK5cT44xdWhKjgkBxwYrgT/<0;1>/*)
```

**What this means:**
- This single descriptor expands into two descriptors:
  1. `wpkh([73c5da0a/84h/0h/0h]xpub6C.../0/*)` - For receiving addresses
  2. `wpkh([73c5da0a/84h/0h/0h]xpub6C.../1/*)` - For change addresses
- Allows defining both receiving and change address chains in a single descriptor
- Particularly useful in wallet software that needs to track both chains

## Practical Usage Guide

### Calculating Descriptor Checksums

You can calculate checksums using Bitcoin Core:

```
bitcoin-cli getdescriptorinfo "wpkh(xpub6Cx5tvq6nACSLJdra1A6WjqTo1SgeUZRFqsX5ysEtVBMwhCCRa4kfgFqaT2o1kwL3esB1PsYr3CUdfRZYfLHJunNWUABKftK2NjHUtzDms2/0/*)"
```

This returns:
```json
{
  "descriptor": "wpkh(xpub6Cx5tvq6nACSLJdra1A6WjqTo1SgeUZRFqsX5ysEtVBMwhCCRa4kfgFqaT2o1kwL3esB1PsYr3CUdfRZYfLHJunNWUABKftK2NjHUtzDms2/0/*)#2tskja72",
  "checksum": "2tskja72",
  "isrange": true,
  "issolvable": true,
  "hasprivatekeys": false
}

```

### Deriving Addresses from Descriptors

You can derive addresses from a descriptor using the `deriveaddresses` command:

```
bitcoin-cli deriveaddresses "tr(xpub6Cx5tvq6nACSLJdra1A6WjqTo1SgeUZRFqsX5ysEtVBMwhCCRa4kfgFqaT2o1kwL3esB1PsYr3CUdfRZYfLHJunNWUABKftK2NjHUtzDms2/100)#5p2mg7zx"
```

This returns the address corresponding to the descriptor. For a range descriptor, you can specify a range:

```
bitcoin-cli deriveaddresses "wpkh(xpub6Cx5tvq6nACSLJdra1A6WjqTo1SgeUZRFqsX5ysEtVBMwhCCRa4kfgFqaT2o1kwL3esB1PsYr3CUdfRZYfLHJunNWUABKftK2NjHUtzDms2/0/*)#2tskja72" "[0,2]"
```

This would return the first three addresses (indexes 0, 1, and 2) from the descriptor.

### Scanning the Blockchain for Descriptor Activity

You can use the `scantxoutset` command to scan the UTXO set for any unspent outputs matching a descriptor:

```
bitcoin-cli scantxoutset "start" '["wpkh(xpub6Cx5tvq6nACSLJdra1A6WjqTo1SgeUZRFqsX5ysEtVBMwhCCRa4kfgFqaT2o1kwL3esB1PsYr3CUdfRZYfLHJunNWUABKftK2NjHUtzDms2/0/*)#2tskja72"]'
```

This is useful for quickly finding funds associated with a descriptor without importing it to your wallet.

### Creating a Descriptor from an Address

You can get the descriptor for an address that belongs to your wallet using `getaddressinfo`:

```
bitcoin-cli getaddressinfo "bc1q0ht9tyks4vh7p5p904t340cr9nvahy7u3re7zz"
```

The response includes a `desc` field containing the descriptor for this address, but only if the address belongs to your wallet. For external addresses, you'll get basic information but no descriptor.


### Importing Descriptors Using Bitcoin Core CLI

#### Creating a descriptor wallet:

```bash
bitcoin-cli createwallet "desc_wallet" true true "" true true true
```

Parameters meaning:
1. Wallet name: "desc_wallet"
2. Disable private keys: true (for watch-only wallet, false otherwise)
3. Blank: true (create a new wallet)
4. Passphrase: "" (no passphrase)
5. Avoid address reuse: true
6. Descriptor wallet: true (important!)
7. Load on startup: true

#### Importing a descriptor to an existing wallet:

```bash
bitcoin-cli -rpcwallet=desc_wallet importdescriptors '[{"desc": "wpkh(xpub.../0/*)#checksum", "timestamp": "now", "range": [0, 100], "internal": false, "label": "Receiving addresses"}]'
```

Parameters explained:
- `desc`: The descriptor with checksum
- `timestamp`: When to start scanning blockchain (use "now" or UNIX timestamp)
- `range`: Address range to import [start, end]
- `internal`: Whether these are change addresses (true) or receiving addresses (false)
- `label`: Optional label for the addresses

#### Example for importing both external and internal descriptors:

```bash
bitcoin-cli -rpcwallet=desc_wallet importdescriptors '[
  {"desc": "wpkh(xpub.../0/*)#checksum1", "timestamp": "now", "range": [0, 100], "internal": false, "label": "Receiving"},
  {"desc": "wpkh(xpub.../1/*)#checksum2", "timestamp": "now", "range": [0, 100], "internal": true, "label": "Change"}
]'
```

## Complex Descriptor Examples

### Miniscript and Policy

Since Bitcoin Core v24, descriptors can be enhanced with Miniscript, a language that allows expressing Bitcoin Script conditions in a structured way. This enables more complex spending conditions while maintaining analyzability. Bitcoin Core v25 added signing support for Miniscript descriptors, and v26 added support for Miniscript expressions in Taproot descriptors.

Example of a 2-of-3 multisig with timelock escape clause using Miniscript within a descriptor:

```
wsh(or_d(multi(2,xpub1.../0/*,xpub2.../0/*,xpub3.../0/*),and_v(v:older(1000),pk(xpub4.../0/*))))#checksum
```

This allows spending either:
- With 2-of-3 signatures from the primary keys, OR
- With a signature from the backup key after 1000 blocks

### Descriptor Templates

For wallet implementations, it's common to use descriptor templates. For example, a wallet might support:

```javascript
function createWalletDescriptors(xpub, walletType) {
  switch(walletType) {
    case 'legacy':
      return {
        external: `pkh(${xpub}/0/*)`,
        internal: `pkh(${xpub}/1/*)`
      };
    case 'segwit':
      return {
        external: `wpkh(${xpub}/0/*)`,
        internal: `wpkh(${xpub}/1/*)`
      };
    // etc.
  }
}
```


### Hardware Wallet Integration

When integrating a hardware wallet with Bitcoin Core:

```bash
# Get xpub from Trezor for account 0
xpub=$(trezorctl get-public-node -n "m/84'/0'/0'" | grep xpub | awk '{print $2}')

# Generate receiving descriptor
receiving_desc=$(bitcoin-cli getdescriptorinfo "wpkh([trezor/84h/0h/0h]$xpub/0/*)" | jq -r '.descriptor')

# Generate change descriptor
change_desc=$(bitcoin-cli getdescriptorinfo "wpkh([trezor/84h/0h/0h]$xpub/1/*)" | jq -r '.descriptor')

# Import both to Bitcoin Core
bitcoin-cli -rpcwallet=trezor_watch importdescriptors "[
  {\"desc\": \"$receiving_desc\", \"timestamp\": \"now\", \"range\": [0, 100], \"internal\": false},
  {\"desc\": \"$change_desc\", \"timestamp\": \"now\", \"range\": [0, 100], \"internal\": true}
]"
```

### Multisignature Wallet Setup

Setting up a 2-of-3 multisig wallet with Bitcoin Core:

```bash
# Create three wallets for key generation
for i in 1 2 3; do
  bitcoin-cli createwallet "key$i" true false "" false false false
done

# Get an xpub from each wallet
xpub1=$(bitcoin-cli -rpcwallet=key1 getnewaddress "" "bech32" | bitcoin-cli -rpcwallet=key1 getaddressinfo | jq -r '.desc' | sed 's/^wpkh(\[\w\+\]//;s:/0/0).*::')
xpub2=$(bitcoin-cli -rpcwallet=key2 getnewaddress "" "bech32" | bitcoin-cli -rpcwallet=key2 getaddressinfo | jq -r '.desc' | sed 's/^wpkh(\[\w\+\]//;s:/0/0).*::')
xpub3=$(bitcoin-cli -rpcwallet=key3 getnewaddress "" "bech32" | bitcoin-cli -rpcwallet=key3 getaddressinfo | jq -r '.desc' | sed 's/^wpkh(\[\w\+\]//;s:/0/0).*::')

# Create multisig descriptor
multisig_desc="wsh(multi(2,$xpub1/0/*,$xpub2/0/*,$xpub3/0/*))"
change_desc="wsh(multi(2,$xpub1/1/*,$xpub2/1/*,$xpub3/1/*))"

# Get checksums
recv_desc=$(bitcoin-cli getdescriptorinfo "$multisig_desc" | jq -r '.descriptor')
chg_desc=$(bitcoin-cli getdescriptorinfo "$change_desc" | jq -r '.descriptor')

# Create and import to multisig wallet
bitcoin-cli createwallet "multisig" true true "" false true true
bitcoin-cli -rpcwallet=multisig importdescriptors "[
  {\"desc\": \"$recv_desc\", \"timestamp\": \"now\", \"range\": [0, 100], \"internal\": false},
  {\"desc\": \"$chg_desc\", \"timestamp\": \"now\", \"range\": [0, 100], \"internal\": true}
]"
```

### Working with Private Keys in Descriptors

For wallets with private keys, you can use WIF format or xprv:

```
# Legacy private key wallet
pkh(5KYZdUEo39z3FPrtuX2QbbwGnNP5zTd7yyr2SC1j299sBCnWjss)

# HD wallet with private key
wpkh(xprv9s21ZrQH143K3QTDL4LXw2F7HEK3wJUD2nW2nRk4stbPy6cq3jPPqjiChkVvvNKmPGJxWUtg6LnF5kejMRNNU3TGtRBeJgk33yuGBxrMPHi/84h/0h/0h/0/*)
```

Note: Never share descriptors containing private keys!

