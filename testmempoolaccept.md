# bitcoin-cli testmempoolaccept

## Overview

The `testmempoolaccept` command allows you to test if multiple raw transactions would be accepted by the mempool without actually broadcasting them to the network. This is extremely useful for parent and child transactions.

## Syntax

```
testmempoolaccept ["rawtx",...] ( maxfeerate )
```

https://developer.bitcoin.org/reference/rpc/testmempoolaccept.html

Parameters:
1. `rawtx` - Array of raw transactions (required, as strings)
   - Each transaction must be a hex-encoded string representing a serialized transaction
   - Transactions can be interdependent (e.g., child transactions can spend outputs from parent transactions in the same batch)

2. `maxfeerate` - Optional parameter to reject transactions with fee rate higher than specified value (in BTC/kB)
   - Useful for preventing accidental high fees
   - Default is 0.10 BTC/kB (extremely high, designed as a safety measure)

## Example

This example tests two interdependent transactions where the second transaction (child) spends an output created by the first transaction (parent):
```
bitcoin-cli testmempoolaccept '["020000000001012f964f42397021f91b20ab513280d2414b7865f1513754bf2780cd39786fcaca8d00000000ffffffff0240420f000000000022002063d1de244bb90c4d46e80da4075fd7dea7b0dfc8173ff725ffd4200f12d65db5bba27b00000000001600142fbbb8a20460e83da09b1a02dc62b0080fcc88a402493046022100efe668d35208f23724a4e879477f2c476db3e366c704782d572874543a6f8b0c022100a0e93af1871d99690d11f60e842d350c226a091bd77a83583aa0a3e98f7127e0012103062b4ace754d8fc40125f6407dc6b5c55ac079e0ec6e6c88022457c9f562340900000000,
02000000000101a9a90b601c6146d3a1759e693df6bd2db3e77e15fef303561c44484fdd63cfd10000000000ffffffff020000000000000000186a1641647973202d2049206b6e6f77206b756e6720667521583e0f00000000001600142fbbb8a20460e83da09b1a02dc62b0080fcc88a40400493046022100945969d5913dcdfddacbb8fb526d1d05a1c6f749d6484137ea6ac2995abc56dd022100971b527e1005e19271e68313b41d796abbbfe40291e4ad277f251305c028912601493046022100b55946499f2b2a9e734d9a55a538300cb336f8014f41a9108b88257a13ff8749022100907a3564417cd5029e8d4767ad97b88b66e124bb0917b0f7eee04e1215b286070147522102a693f085c8f3cbf9e109a8d9dda9cd6805e2e8c1b82d9abf64f785a337dbe3ff210227b6997a65ef7a576f565bf29e889a71000d5a20260a179ddd2d173d7fd165ce52ae00000000"]'
```
## Response

```
[
  {
    "txid": "d1cf63dd4f48441c5603f3fe157ee7b32dbdf63d699e75a1d346611c600ba9a9",
    "wtxid": "33b3a44481b018a6c5e71eb51e03f85563abdb3ff80950d48c712c7487245f06",
    "allowed": true,
    "vsize": 153,
    "fees": {
      "base": 0.00001000,
      "effective-feerate": 0.00006535,
      "effective-includes": [
        "33b3a44481b018a6c5e71eb51e03f85563abdb3ff80950d48c712c7487245f06"
      ]
    }
  },
  {
    "txid": "06f949da3dd721de560936ba431c1531ee1403e0955b840724f675a8085e373c",
    "wtxid": "0c7698816e09b8f9bef41e17ce6f4da29583701a039f941d1eb28f776ec5a5a9",
    "allowed": true,
    "vsize": 171,
    "fees": {
      "base": 0.00001000,
      "effective-feerate": 0.00005847,
      "effective-includes": [
        "0c7698816e09b8f9bef41e17ce6f4da29583701a039f941d1eb28f776ec5a5a9"
      ]
    }
  }
]
```

## Response Fields Explained

- **txid**: The transaction identifier (hash of the transaction without witness data)
- **wtxid**: The witness transaction identifier (hash including witness data)
- **allowed**: Boolean indicating whether the transaction would be accepted into the mempool
- **vsize**: Virtual size of the transaction in virtual bytes (weight units ÷ 4)
- **fees**:
  - **base**: The transaction fee in BTC
  - **effective-feerate**: The effective fee rate in BTC/kB, accounting for ancestor/descendant relationships
  - **effective-includes**: List of transaction IDs included in the effective fee calculation

## Common Use Cases

1. **Fee estimation verification**: Ensure transactions have sufficient fees before broadcasting, preventing stuck transactions
2. **RBF (Replace-By-Fee) testing**: Verify replacement transactions meet policy requirements for fee bumping
3. **CPFP (Child-Pays-For-Parent) validation**: Test if child transactions properly incentivize mining of parent transactions
4. **Complex script validation**: Test if transactions with custom scripts will be accepted by network rules
5. **Transaction chain validation**: Verify that a sequence of dependent transactions is valid when submitted together
