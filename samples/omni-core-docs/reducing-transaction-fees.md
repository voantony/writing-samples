Reducing transaction fees
=========================

Every Omni Layer transaction is a Bitcoin transaction, so it pays the normal Bitcoin fee: its size in virtual bytes (vB) times the fee rate in satoshi per vB. This page lists what you can do today to keep both numbers low, and what will help once more features are activated.

Sizes below are estimates for a simple send with one input, one Class C (OP_RETURN) payload, the receiver output and change. Each extra input adds the input size again.

| Sender address type                | Example      | Simple send | Extra input | Allowed on mainnet      |
|------------------------------------|--------------|-------------|-------------|-------------------------|
| Pay-to-pubkey-hash (P2PKH)         | `1...`       | ~257 vB     | ~148 vB     | yes                     |
| P2SH-wrapped SegWit (P2SH-P2WPKH)  | `3...`       | ~197 vB     | ~91 vB      | yes                     |
| Native SegWit (P2WPKH)             | `bc1q...`    | ~172 vB     | ~68 vB      | not yet (see below)     |
| Taproot (P2TR)                     | `bc1p...`    | ~185 vB     | ~58 vB      | not yet (see below)     |

## What you can do today

### 1. Send from a `3...` (P2SH-SegWit) address

A P2SH-wrapped SegWit address makes a simple send about 23% smaller than a `1...` address, and each extra input about 40% smaller. Create one with:

```bash
$ omnicore-cli getnewaddress "" "p2sh-segwit"
```

Move your tokens to it once with `omni_send` or `omni_sendall`, and send from it afterwards.

**Do not use native SegWit (`bc1q...`) or Taproot (`bc1p...`) addresses for Omni Layer tokens on mainnet yet.** These script types are not activated for the Omni Layer on mainnet. A transaction sent from such an address is invalid, and an output to such an address is ignored by the Omni Layer, so the tokens do not arrive there. On testnet and regtest they are already allowed.

### 2. Use few inputs, and consolidate when fees are low

Omni Layer balances belong to addresses, not to individual coins. The bitcoin needed for fees, however, is spent coin by coin, and every coin used as input makes the transaction larger. If a sending address holds many small coins, combine them into one in a plain bitcoin transaction from that address to itself while fees are low. This does not move any tokens.

Alternatively, `omni_funded_send` and `omni_funded_sendall` (see below) spend all bitcoin of the sender in the same transaction.

### 3. Choose the fee rate and the time

Omni Core creates transactions with the wallet's fee settings, so the usual Bitcoin Core options apply:

- `-txconfirmtarget=<n>`: target confirmation within `n` blocks when estimating the fee. A higher value usually means a lower fee.
- `settxfee <amount>` or `-paytxfee=<amount>`: use a fixed fee rate (BTC per kvB) instead of the estimate.

Bitcoin fee rates change a lot during a day and a week. Transactions that are not urgent are much cheaper when the mempool is quiet, often at 1 to 3 sat/vB. Your node will not create transactions below its minimum relay fee (`-minrelaytxfee`).

### 4. Let a separate address pay the fees

`omni_funded_send` and `omni_funded_sendall` take a `feeaddress`. All bitcoin of the sender is spent, missing bitcoin is taken from the fee address, and the change goes to the fee address.

```bash
$ omnicore-cli "omni_funded_send" "1DFa5bT6KMEr6ta29QJouainsjaNBsJQhH" \
    "15cWrfuvMxyxGst2FisrQcvcpF48x6sXoH" 1 "100.0" \
    "15Jhzz4omEXEyFKbdcccJwuVPea5LqsKM1"
```

This is useful for services with many token addresses: the token addresses do not need to hold bitcoin, and the bitcoin of one address pays for many transfers.

### 5. Keep the reference amount at the minimum

The receiver of a simple send gets a small bitcoin output (the "reference output"). By default Omni Core uses the smallest allowed amount, for example 546 satoshi for a `1...` receiver. Only set `referenceamount` in `omni_send` if you have a reason to.

### 6. Keep Class C (OP_RETURN) encoding enabled

Omni Core stores the transaction data in an OP_RETURN output (Class C). If the data does not fit, or `-datacarrier=0` is set, it falls back to Class B, which stores the data in bare multisig outputs. Class B transactions are larger and leave bitcoin in those outputs. Keep the default `-datacarrier=1`.

## What will help once activated

### Send-to-many (batching)

The send-to-many transaction (type 7, `omni_sendtomany`) pays several receivers in one transaction. It is implemented, but not yet activated on mainnet. Estimated sizes with a `3...` sender:

| Receivers | One send-to-many | Separate simple sends | Saving |
|-----------|------------------|-----------------------|--------|
| 7         | ~445 vB          | ~1379 vB              | ~68%   |
| 10        | ~568 vB          | ~1970 vB              | ~71%   |
| 50        | ~2208 vB         | ~9850 vB              | ~78%   |

Each receiver adds 9 bytes to the payload. With the default `-datacarriersize` of 83 bytes, up to 7 receivers fit into one OP_RETURN output. More receivers need a larger `-datacarriersize`, and the network must relay and mine such transactions.

### Native SegWit and Taproot

Once native SegWit and Taproot are activated for the Omni Layer on mainnet, `bc1q...` addresses will make a simple send about 13% smaller than a `3...` address, and extra inputs smaller still.
