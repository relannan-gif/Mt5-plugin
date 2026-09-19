# OneRoyal — MT5 Account-Level Restriction Plugin

Technical specification for an MT5 server-side plugin that restricts individual trading accounts
**without** moving accounts between groups, changing group permissions, or modifying instrument or
symbol settings.

📄 **[Technical Specification](docs/OneRoyal-MT5-Account-Restriction-Plugin-Technical-Specification.md)**

## Status

**DRAFT v0.1 — evidence-incomplete. Not approved for production release.**

The four mandatory source materials (`MetaTrader5SDK.chm`, `API.zip`, `Manager.zip`, and the
*Creating a Simple Plugin — Server API* PDF) **were not supplied and have not been read.**

Consequently the specification contains **zero SDK citations**. Every statement that would normally
rest on a header declaration or documentation topic is labelled `Requires SDK verification` and
carries a verification instruction in Appendix B instead of a citation. No code has been written,
compiled or tested; all tests in §14 are marked **NOT RUN**.

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

## Two findings that drive the programme

1. **Funding cannot be controlled inside MT5.** A plugin can refuse a ledger entry; it cannot stop a
   PSP charging a card or a bank releasing a payout. An MT5 rejection *after* the charge produces a
   stranded payment — worse than allowing it. The authoritative funding control must be a
   **pre-charge gate in the CRM/payment layer** (§7). The plugin is a backstop.

2. **The trading rule is decidable and buildable today.** The decision logic (§5) depends only on
   position state and requested volume, uses integer arithmetic only, and has no SDK dependency —
   so it can be implemented and unit-tested before the SDK arrives.

## Recommended sequence

| # | Action | Blocked? |
|---|---|---|
| 1 | Obtain the SDK; complete the Appendix B worksheet (22 open rows) | Needs SDK access |
| 2 | Start the CRM/payment gate design — longest lead time, not SDK-dependent | **No** |
| 3 | Build the policy engine (§5) as a standalone tested library | **No** |
| 4 | Build the SDK adapter once §6 is answered | Yes — on step 1 |

Do not begin plugin coding against assumed interfaces.

## Release blockers

Eleven are open (§16). The two most commercially significant:

- **BLK-02** — funding prevention is not an MT5 capability; every production payment writer needs a
  traced pre-charge veto.
- **BLK-04** — privileged execution paths may bypass trade hooks entirely; needs a vendor answer.

## Repository layout

```
docs/   Technical specification (the deliverable)
```

Source directories follow the proposed project tree in §8.6 once WP-00/WP-01 begin.
Licensed SDK files and production credentials must never be committed here (REQ-AR-06).
