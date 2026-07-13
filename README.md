# Ledger Bitcoin Cash II Application

Ledger device application for Bitcoin Cash II (BCH2).

This is a clone of Ledger's [app-bitcoin-legacy](https://github.com/LedgerHQ/app-bitcoin-legacy),
adding Bitcoin Cash II as a coin variant. BCH2 is a Bitcoin Cash protocol chain: it uses the
same `SIGHASH_FORKID` (BIP-143) signing scheme, CashAddr addresses, and P2PKH/P2SH script types
as Bitcoin Cash, with its own CashAddr prefix and application identity.

## Changes from app-bitcoin-legacy

The diff against upstream is intentionally small so it can be reviewed quickly:

- **Makefile** — a `bitcoin_cash_ii` variant: BIP-44 coin type 145, P2PKH version 0, P2SH version 5,
  `COIN_KIND_BITCOIN_CASH_II`, and `COIN_CASHADDR_PREFIX="bitcoincashii"`.
- **[lib-app-bitcoin](https://github.com/BitcoincashII/lib-app-bitcoin) (branch `bch2`)** — adds the
  `COIN_KIND_BITCOIN_CASH_II` coin kind, which reuses Bitcoin Cash's forkId signing and CashAddr
  display paths, and parameterizes the CashAddr prefix (defaulting to `bitcoincash`, so upstream
  behaviour is unchanged).
- **Icons** — Bitcoin Cash II device icons and glyphs.

BCH2 does not use SegWit. As with the Bitcoin Cash application, the app's internal "segwit" code path
is the BIP-143 sighash machinery used to compute the forkId signature; the transactions it produces
are standard non-SegWit P2PKH.

## Parameters

| | |
|---|---|
| Ticker | BCH2 |
| BIP-44 coin type | 145 |
| Derivation | `m/44'/145'` |
| Address format | CashAddr (`bitcoincashii:`) and legacy Base58 |
| Signing | ECDSA, `SIGHASH_ALL \| SIGHASH_FORKID` (0x41) |

## Build

```
BOLOS_SDK=$NANOSP_SDK make COIN=bitcoin_cash_ii
```

Substitute `$NANOX_SDK`, `$STAX_SDK`, `$FLEX_SDK`, or `$APEX_P_SDK` for other devices. The
[ledger-app-builder](https://github.com/LedgerHQ/ledger-app-builder) Docker image provides the SDKs.

## Tests

Functional tests use the [Speculos](https://github.com/LedgerHQ/speculos) emulator through the Ragger
framework. See [tests/README.md](tests/README.md).

## Are you developing a Ledger device application?

- See the developers' documentation on the [Developer Portal](https://developers.ledger.com/)
- [Go on Discord](https://developers.ledger.com/discord-pro/) to chat with developer support and the
  developer community.

## Client Library

Include the necessary headers (copied from the js/ directory) in your web page

```html
<head>
  <script src="thirdparty/q.js"></script>
  <script src="thirdparty/async.min.js"></script>
  <script src="thirdparty/u2f-api.js"></script>
  <script src="dist/ledger-btc.js"></script>
</head>
```

Create a communication object

```javascript
var dongle = new LedgerBtc(20);
```

For each UTXO included in your transaction, create a transaction object from the raw serialized version of the transaction used in this UTXO

```javascript
var tx1 = dongle.splitTransaction("01000000014ea60aeac5252c14291d428915bd7ccd1bfc4af009f4d4dc57ae597ed0420b71010000008a47304402201f36a12c240dbf9e566bc04321050b1984cd6eaf6caee8f02bb0bfec08e3354b022012ee2aeadcbbfd1e92959f57c15c1c6debb757b798451b104665aa3010569b49014104090b15bde569386734abf2a2b99f9ca6a50656627e77de663ca7325702769986cf26cc9dd7fdea0af432c8e2becc867c932e1b9dd742f2a108997c2252e2bdebffffffff0281b72e00000000001976a91472a5d75c8d2d0565b656a5232703b167d50d5a2b88aca0860100000000001976a9144533f5fb9b4817f713c48f0bfe96b9f50c476c9b88ac00000000");

var tx2 = dongle.splitTransaction("...")
```

To sign a transaction involving standard (P2PKH) inputs, call createPaymentTransactionNew_async with the folowing parameters

 - `inputs` is an array of [ transaction, output_index, optional redeem script, optional sequence ]
 - `associatedKeysets` is an array of BIP 32 paths pointing to the path to the private key used for each UTXO
 - `changePath` is an optional BIP 32 path pointing to the path to the public key used to compute the change address
 - `outputScript` is the hexadecimal serialized outputs of the transaction to sign
 - `lockTime` is the optional lockTime of the transaction to sign, or default (0)
 - `sigHashType` is the hash type of the transaction to sign, or default (all)

This method returns the signed transaction ready to be broadcast

```javascript
dongle.createPaymentTransactionNew_async(
   [ [tx, 1] ],
   ["0'/0/0"],
   undefined,
   "01905f0100000000001976a91472a5d75c8d2d0565b656a5232703b167d50d5a2b88ac").then(
     function(result) { console.log(result);}).fail(
     function(error) { console.log(error); });
);
```
