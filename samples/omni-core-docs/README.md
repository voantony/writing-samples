# Omni Core documentation (volunteer)

These pages are documentation I wrote against the Omni Core source in October 2026, as unpaid volunteer work. They are not official Omni Layer documentation, and they had not been merged when this portfolio was published.

Omni Core is an open-source Bitcoin client with Omni Layer support. I was a founding board member of the Mastercoin Foundation (later Omni) from 2013 to 2016. The writing here is recent documentation work, not paid employment.

- [Reducing transaction fees](reducing-transaction-fees.md) — a user guide I added, with estimated sizes by address type, what works on mainnet today, and a warning not to use `bc1…` addresses for Omni tokens until those types are activated.
- [Debug log categories](debug-log-categories.md) — a reference I added by reading `InitDebugLogLevels()` and the `msc_debug_*` flags in the source. It lists every `-omnidebug` category, the default, what it actually logs, and which names are accepted but unused.
- [`redeemaddress` applies only to Class B](redeemaddress.md) — a correction to the RPC docs. The old text said `redeemaddress` was the address that could spend the transaction dust. It is only used for Class B (bare multisig) data outputs; for the default Class C encoding it is validated but has no effect.

I also proposed related build and test fixes in the same stretch of work (a missing `<cstdint>` include that broke GCC 13, and a few test/startup-error fixes). Those are code changes, not included here.
