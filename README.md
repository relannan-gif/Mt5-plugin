# OneRoyal — MT5 Account-Level Restriction Plugin

Technical specification for an MT5 server-side plugin that restricts individual trading accounts
**without** moving accounts between groups, changing group permissions, or modifying instrument or
symbol settings.

📄 **[Technical Specification](docs/OneRoyal-MT5-Account-Restriction-Plugin-Technical-Specification.md)**

## Status

**DRAFT v0.2 — SDK-evidenced. Not approved for production release.**

Reviewed against **MetaTrader 5 SDK 3.1 — Server API version 6182 (5 Sep 2026)**: the C++ headers,
the server/manager examples, and the documentation unpacked from `MetaTrader5SDK.chm`. Every
SDK-derived statement is cited as `file:line` or `CHM: topic.htm`. The *Creating a Simple Plugin*
PDF is not shipped in SDK 3.1; the reference `ServerPlugin` example and the CHM Server API section
were used in its place.

No code has been compiled and no test has been run — no MT5 test server, gateway or payment sandbox
was available. All §14 tests are marked **NOT RUN**. 17 of 22 SDK verification rows are answered
from the documentation; 5 need a live server.

## What the restriction does

| Operation on a restricted account | Outcome |
|---|---|
| Deposit / withdraw / transfer | **Prevent** (transfers: both endpoints) |
| Open a new position | **Prevent** |
| Increase position volume | **Prevent** |
| Open an opposing hedge | **Prevent** |
| Reverse into opposite exposure | **Prevent** |
| Fully or partially close a position | **Permit** |
| Modify Stop Loss / Take Profit | **Permit** |

## What the SDK settled

- **Trading enforcement has documented, rejectable hooks.** `HookTradeRequestAdd` runs after all
  server checks and before the order exists; `HookTradeRequestProcess` runs immediately before
  execution. Server-generated SL/TP, stop-out and pending-activation requests traverse the same hooks.
- **Funding still cannot be controlled inside MT5.** Only two funding writers reach a hook
  (`TA_DEALER_BALANCE` requests and terminal `TA_TRANSFER`), and only the first hook. The direct
  Manager balance methods, the Web API balance command and gateway synchronisation have no documented
  hook. The authoritative control must be a **pre-charge gate in the CRM/payment layer** (§7).
- **There is no atomic Manager/Web API transfer.** A CRM transfer is two balance operations;
  the only atomic transfer is the terminal's, and it is same-server only.
- **A privileged trading bypass is documented.** `DealPerform*` creates no request and applies no
  routing; only administrative control and post-event detection cover it.
- **Volume units, transfer field semantics, plugin-parameter limits and the threading model** are
  now documented facts rather than open questions (§5.2, §6.7-d, §9.6, §10.5).

## Recommended sequence

| # | Action | Blocked? |
|---|---|---|
| 1 | Procure an MT5 test server; close the 5 open Appendix B rows | Test environment |
| 2 | Start the CRM/payment pre-charge gate design — longest lead time, not SDK-dependent | **No** |
| 3 | Build the policy engine (§5) as a standalone tested library | **No** |
| 4 | Build the SDK adapter against the real headers (Appendix C) | **No** |

## Release blockers

Twelve remain open (§16). The commercially significant ones: **BLK-02** (funding is not an MT5
capability), **BLK-04** (privileged `DealPerform*` bypass — administrative), **BLK-08** (direct
balance methods — undocumented hook traversal; vendor question drafted).

## Repository layout

```
docs/   Technical specification (the deliverable)
```

Source directories follow the proposed project tree in §8.6 once WP-01/WP-02 begin.
Licensed SDK files and production credentials must never be committed here (REQ-AR-06).
