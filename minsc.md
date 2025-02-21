# Transaction Info and Sighash Using Minsc

## **Introduction**

[Minsc](https://github.com/shesek/minsc) is a powerful tool for Bitcoin scripting, developed by [Shesek](https://github.com/shesek/).
You can do many amazing things with Minsc, like [CTV-based Vault](https://min.sc/v0.3/#github=examples/ctv-vault.minsc) and [HTLC](https://min.sc/v0.3/#github=examples/htlc.minsc).
In this guide we will see how to check transaction details and compute  sighash using Minsc.\
\
We will use two simplified minsc templates for [p2wpkh](https://min.sc/v0.3/#gist=4f3a74b78978ea2a650b96f5a72352ce) and [p2wsh](https://min.sc/v0.3/#gist=f677ffb2aec20eee0da9d64417695d4e).


Use Minsc for **testnet only!**

---

## **1. Set Up Your Variables**

We will define private key and output details. We can use any form of key, but in our example we will use an extended private key. 



Open p2wpkh template in the browser:

[https://min.sc/v0.3/#gist=4f3a74b78978ea2a650b96f5a72352ce](https://min.sc/v0.3/#gist=4f3a74b78978ea2a650b96f5a72352ce)

Declare your private key:

```minsc
$tprv = tprv8ZgxMBicQKsPe2ueBRfzTzgpt7gnRZdwo4rEPKsLmmpGHQxZscNKv45nsgfohcoWgturdL3c2J7FakV8adSk3huTA1RSWJEXFukwkJWF3gC;
```

###

Your bip32 key derivation index:

```
$derivationIndex = 0;
```

\
Your output address:

```
$outputAddr = tb1qwtcm09rg8eez7ep0x4tkqkrtasa0mytufw7xzz;
```

\
And the amount of the output:

```
$amount = 0.85208938 BTC;
```


* We can use [mempool.space api](https://mempool.space/signet/api/tx/138462fe0acc6ff1b6aefaf61b4c92db04fc340ad6b56eaee6b67e812e07ba57) to get the amount and address of your output.

\
We will add an assert to ensure that your derived address matches the expected output. This step is crucial because it verifies that the private key derivation is correct and aligns with the expected output address, preventing potential errors when signing transactions. Note the derivation path is `84h/1h/0h/0/derivationIndex`, adjust it to your needs.

```minsc
assert::eq(address(wpkh($tprv/84h/1h/0h/0/$derivationIndex)), $outputAddr);
```

---

## **2. Load the Transaction**

Define the transaction using the `tx()` function.

```minsc
$tx = tx(0x020000000001012dbbe91eceb76830221374c1a0d0080fc88f01ad2cc121b07bb06b0a27cb3abff200000000ffffffff0240420f0000000000220020e2f55cdc156913d6fc86f4ab684d54c113a1ad04d7cdec2f8f706a5bc64edc78d5a27b00000000001600147022f7802b1c8f495b191e014849a217de3235f802483045022100a905549b19b5ef3563bda630d3050aa318d56dd7101ce647eff5bba3b7d947660220213c1c9890410374a6271bc69e48e94ea4ea4f6bd8cdfc375d29c9239edf7f930121039c66ff2131622270d5eee20cd45fac7457912b16704d1cd17156957771c18a1400000000);
```

This represents your raw Bitcoin transaction.

---

## **3. Remove Witness Data**

Since signatures cover transaction data without witnesses, create a witness-stripped transaction:

```minsc
$tx_no_witness = tx::with_witness($tx, [ 0: [] ]);
```

---

## **4. Compute the Sighash**

The sighash is a hash of the transaction data that must be signed. Compute it using:

```minsc
$sighash = tx::sighash($tx_no_witness, 0, [ $outputAddr:$amount ], SIGHASH_ALL);
```

This command:

- Targets input index `0`.
- Uses the given output address and amount.
- Specifies `SIGHASH_ALL` (signing the entire transaction).

---

## **5. Generate the Signature**

Sign the transaction hash using the derived private key:

```minsc
$signature = ecdsa::sign($tprv/84h/1h/0h/0/$derivationIndex, $sighash) + SIGHASH_ALL;
```

---

## **6. Full Template Example (p2wpkh)**

```minsc
// template for signet p2wpkh

// replace key (test only!), index, output address & amount
// you can find your output address & amount using: https://boss2025.xyz/api/tx/yourtxid
$tprv = tprv8ZgxMBicQKsPe2ueBRfzTzgpt7gnRZdwo4rEPKsLmmpGHQxZscNKv45nsgfohcoWgturdL3c2J7FakV8adSk3huTA1RSWJEXFukwkJWF3gC; 
$derivationIndex = 999;
$outputAddr = tb1qwtcm09rg8eez7ep0x4tkqkrtasa0mytufw7xzz;
$amount = 0.85208938 BTC;

// uncomment this, if assert, verify key and index
//assert::eq(address(wpkh($tprv/84h/1h/0h/0/$derivationIndex)), $outputAddr);

$tx = tx(0x020000000001012dbbe91eceb76830221374c1a0d0080fc88f01ad2cc121b07bb06b0a27cb3abff200000000ffffffff0240420f0000000000220020e2f55cdc156913d6fc86f4ab684d54c113a1ad04d7cdec2f8f706a5bc64edc78d5a27b00000000001600147022f7802b1c8f495b191e014849a217de3235f802483045022100a905549b19b5ef3563bda630d3050aa318d56dd7101ce647eff5bba3b7d947660220213c1c9890410374a6271bc69e48e94ea4ea4f6bd8cdfc375d29c9239edf7f930121039c66ff2131622270d5eee20cd45fac7457912b16704d1cd17156957771c18a1400000000);

$tx_no_witness = tx::with_witness($tx, [ 0: [] ]);

$sighash = tx::sighash($tx_no_witness, 0, [ $outputAddr:$amount ], SIGHASH_ALL);

$signature = ecdsa::sign($tprv/84h/1h/0h/0/$derivationIndex, $sighash) + SIGHASH_ALL;

```



- You can check the official example here, that uses Wif format\
  [https://min.sc/v0.3/#github=examples/manual-signing-p2wpkh.minsc](https://min.sc/v0.3/#github=examples/manual-signing-p2wpkh.minsc)

---

## **7. Expected Output (p2wpkh)**

```json
// Defined variables:

$tprv = tprv8ZgxMBicQKsPe2ueBRfzTzgpt7gnRZdwo4rEPKsLmmpGHQxZscNKv45nsgfohcoWgturdL3c2J7FakV8adSk3huTA1RSWJEXFukwkJWF3gC // seckey

$derivationIndex = 999 // int

$outputAddr = tb1qwtcm09rg8eez7ep0x4tkqkrtasa0mytufw7xzz // address

$amount = 85208938 // int

$tx = tx [
  "version": 2,
  "inputs": [
    [
      "prevout": bf3acb270a6bb07bb021c12cad018fc80f08d0a0c17413223068b7ce1ee9bb2d:242,
      "witness": [ 0x3045022100a905549b19b5ef3563bda630d3050aa318d56dd7101ce647eff5bba3b7d947660220213c1c9890410374a6271bc69e48e94ea4ea4f6bd8cdfc375d29c9239edf7f9301, 0x039c66ff2131622270d5eee20cd45fac7457912b16704d1cd17156957771c18a14 ]
    ]
  ],
  "outputs": [
    tb1qut64ehq4dyfadlyx7j4ksn25cyf6rtgy6lx7ctu0wp49h3jwm3uqrw4sqj:0.01 BTC,
    tb1qwq300qptrj85jkcercq5sjdzzl0ryd0cvxk8wh:0.08102613 BTC
  ]
] // transaction

$tx_no_witness = tx [
  "version": 2,
  "inputs": [
    bf3acb270a6bb07bb021c12cad018fc80f08d0a0c17413223068b7ce1ee9bb2d:242
  ],
  "outputs": [
    tb1qut64ehq4dyfadlyx7j4ksn25cyf6rtgy6lx7ctu0wp49h3jwm3uqrw4sqj:0.01 BTC,
    tb1qwq300qptrj85jkcercq5sjdzzl0ryd0cvxk8wh:0.08102613 BTC
  ]
] // transaction

$sighash = 0xdf2bbe5674893a76cfcd86d7099e680105250feabf031301e64cc4523d56ce1f // bytes

$signature = 0x3044022006fb261e2eb52c2adc7495123b868dac3ff6623eb5753ef34f3ddd3600c8423002201da932213cb2dd037ac787a461a6042dbb9f8287526f95874f6dad6c9611501c01 // bytes
```

```json


```

###



## **8. Full Template Example (p2wsh)**



```
// template for signet p2wsh

// update your key (test only)
$tprv = tprv8ZgxMBicQKsPe2ueBRfzTzgpt7gnRZdwo4rEPKsLmmpGHQxZscNKv45nsgfohcoWgturdL3c2J7FakV8adSk3huTA1RSWJEXFukwkJWF3gC;
$amount = 0.01 BTC;

$tx = tx(0x02000000000101d62234a2ce378e85baad2d0499c60dbc3037b29da2cf4d4e796e52a6fd264d570000000000ffffffff0200e1f50500000000220020421305d3c9f5faee4f8fa7dc79f54c5eb2971ff4819cfac68fcb219c14f8aed5fb8d435500000000160014f1a65dab1d878c0ae057a5d3cc1b7d31f494d774024830450221008e9582cf252ed3b9f05c38a017c675ae8c55f0aba3bc9c1dfddb0fbbe3b17c3f022002d4c28e687597aa89445a78afdee34d96852493c140b573fa8b157edc3a704901210250e42d3f14ed353ffb49c8a041bab867dadbbb987a41007c7424aa7c5cad90ca00000000);


$tx_no_witness = tx::with_witness($tx, [ 0: [] ]);

$sk0 = bytes(derived($tprv/84h/1h/0h/0/0));
$sk1 = bytes(derived($tprv/84h/1h/0h/0/1));
$pk1 = bytes(derived(pubkey($tprv/84h/1h/0h/0/0)));
$pk2 = bytes(derived(pubkey($tprv/84h/1h/0h/0/1)));

$sighash = tx::sighash($tx_no_witness, 0, [ wsh(`2 $pk1 $pk2 2 OP_CHECKMULTISIG`):$amount], SIGHASH_ALL);
```

- This template version is simplfied for learning purpose, and misses the signature part. 
If you need that, check out the official example:\
  [https://min.sc/v0.3/#github=examples/multisig-simple.minsc](https://min.sc/v0.3/#github=examples/multisig-simple.minsc)


## **9. Expected Output (p2wsh)**



```
// Defined variables:

$tprv = tprv8ZgxMBicQKsPe2ueBRfzTzgpt7gnRZdwo4rEPKsLmmpGHQxZscNKv45nsgfohcoWgturdL3c2J7FakV8adSk3huTA1RSWJEXFukwkJWF3gC // seckey

$amount = 1000000 // int

$tx = tx [
  "version": 2,
  "inputs": [
    [
      "prevout": 574d26fda6526e794e4dcfa29db23730bc0dc699042dadba858e37cea23422d6:0,
      "witness": [ 0x30450221008e9582cf252ed3b9f05c38a017c675ae8c55f0aba3bc9c1dfddb0fbbe3b17c3f022002d4c28e687597aa89445a78afdee34d96852493c140b573fa8b157edc3a704901, 0x0250e42d3f14ed353ffb49c8a041bab867dadbbb987a41007c7424aa7c5cad90ca ]
    ]
  ],
  "outputs": [
    tb1qggfst57f7hawunu05lw8na2vt6efw8l5sxw04350evsec98c4m2s9vg9ql:1 BTC,
    tb1q7xn9m2cas7xq4czh5hfucxmax86ff4m5pj7ce8:14.30490619 BTC
  ]
] // transaction

$tx_no_witness = tx [
  "version": 2,
  "inputs": [
    574d26fda6526e794e4dcfa29db23730bc0dc699042dadba858e37cea23422d6:0
  ],
  "outputs": [
    tb1qggfst57f7hawunu05lw8na2vt6efw8l5sxw04350evsec98c4m2s9vg9ql:1 BTC,
    tb1q7xn9m2cas7xq4czh5hfucxmax86ff4m5pj7ce8:14.30490619 BTC
  ]
] // transaction

$sk0 = 0x490e081f444ec00e525a1eab9976505f0e4b0c9e3878c1b05015859adc0deae0 // bytes

$sk1 = 0x7c8a7a16352b924ae807a3ccf19b5b6be453ca3e5263b8fc338ba5e9ba65dd6a // bytes

$pk1 = 0x03df653824ac6a3e8bcca1372e9b36241acc654a2d0cc94c76cb4cf613447391c7 // bytes

$pk2 = 0x03eda350a58a973f31466663b348b8340c94251ee0c5d01cbaeaa5e17e4d6c8e14 // bytes

$sighash = 0xc90832d5fe9abcf36a8be02b7fce0e37e5c2428b865448fb55ee2e5f9cafec17 // bytes
```

---


