# `redeemaddress` applies only to Class B

This is a documentation correction I wrote for Omni Core's RPC reference. The old help text and `rpc-api.md` described `redeemaddress` as the address that can spend the transaction dust. After reading `WalletTxBuilder` and checking the behaviour, that is not what the parameter does.

`redeemaddress`, where accepted, is the address whose public key can redeem Class B (bare multisig) data outputs. Change is always returned to the sender, and the reference output is paid to the receiver, if any. The parameter does not select either of those outputs, and providing it does not force a Class B transaction.

Class C (`OP_RETURN`) is the default whenever the payload fits in the data carrier (`-datacarrier` / `-datacarriersize`) and `OP_RETURN` outputs are allowed. For Class C the parameter is validated, but has no effect. The fixed payloads built by `omni_send`, `omni_sendall`, `omni_sendsto`, and `omni_sendnonfungible` fit in the default data carrier, so `redeemaddress` has no effect on those calls unless Class C is disabled or the data-carrier limit is reduced below the payload size. `omni_sendrawtx` uses the supplied payload and consults `redeemaddress` only when that payload falls back to Class B.

The per-RPC parameter line was updated from:

> an address that can spend the transaction dust (sender by default)

to:

> Class B redeem address (sender by default); validated, but has no effect for Class C

This is volunteer documentation, not official Omni Layer documentation. It has not been merged into Omni Core.
