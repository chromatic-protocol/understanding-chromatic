# `Chromatic-protocol` Specification

## Market
- **Market Opening:**
  - Anyone can create a unique market with an oracle and settlement token combination.
- **Oracle:**
  - Any on-chain feed.
  - `Oracle tier` determined by the protocol DAO.
  - Values determined by `oracle tier`:
    - `Leverage range`:
      - Level 1: `x10`
      - Level 2: `x20`
- **Settlement Token:**
  - Chosen from settlement tokens added by DAO voting.
  - Criteria for adding settlement token:
    - Minimum betting amount.
    - Minimum liquidity deposit/withdrawal amount.
  - Ssettings:
    - Minimum betting amount: about `10$` 
    - Minimum liquidity deposit: about `10$`.

- **Values determined by DAO:**
  - Common to all markets:
    - `Take profit rate range`: Initially set to `10% ~ 1000%`.
    - `Stop loss rate range`: Initially set to `10% ~ 100%`.
  - Specific to each market:
    - `Leverage level`: Oracle-specific leverage range adjustment.
    - `Protocol fee`:
      - Initially set to `0`.
      - Protocol fee rate = 1/N, `4` <= `N` <= `10` (25% ~ 10%).

- **Execution Price:**
  - Entry: Use the price of `the next round`.
  - Taker's exit order: Use the price of `the last round`.
  - Liquidation: Use the price of `the last round`.
- **Taker's Trading Account:**
  - Utilize the taker account contract.
  - Does not trade with EOA (Externally Owned Account) directly.
  - Each EOA can create a single `Account` contract.

## Settlement
- **Entry:**
  - `Trading fee`:
    - Withdrawn separately using a callback for position calculation.
    - Depletes from the lowest fee pool slot.
  - Uses the `interest fee rate` during entry.
- **Liquidation:**
  - Automatic liquidation using a keeper.
    - Collects `liquidation fee` and deducts it from the profitable side.
      - `Liquidation fee` = `Liquidation fee rate` * `abs(trading profit)`.
      - Market contract accumulates `liquidation fee` and pays the keeper fee.
    - `Take profit resolving` condition:
      - `Maker margin` <= `taker's trading profit` - `sum(fees)`.
    - `Stop loss resolving` condition:
      - `Taker margin` <= `taker's trading loss` + `sum(fees)`.
  - **Exit by Taker:**
    - Returns `taker margin` + `trading pnl` - `sum(fees)`.

- **Fees:**
  - `Trading fee`.
  - `Interest fee`.
  - `Protocol fee`:
    - Part of the fee received by the maker from the taker, similar to Uniswap.
    - Rate:
      - Default `0`.
      - `1/N`: `4 <= N <= 10`.
      - Determined by the protocol DAO.
        - Applies from the entry of positions after implementation.
  - `Keeper fee`.

## Liquidity
- **Set trade fees for bins:**
    - range is determined by observed historical data of CEX
    - `0.01% ~ 0.09%` (9 slots, `0.01%` increments).
    - `0.1% ~ 0.9%` (9 slots, `0.1%` increments).
    - `1% ~ 9%` (9 slots, `1%` increments).
    - `10% ~ 50%` (9 slots, `5%` increments).
- **Valuation:**
  - Calculated using the price at `the next round`.
  - Includes unrealized PnL.
- **maker certificates - `CLB` token:**
  - `ERC1155`.
    - Fungible within the same `bin`.
    - Long/short distinguished by offset
- **Withdrawal:**
  - Processed from the available unused margin in the liquidity.

## Miscellaneous
- **Currently Unsupported:**
  - Limit orders.
  - Partial liquidation.
  - Additional position openings (separate entry).
  - Adjustment of margin rates (adding/withdrawing margin considerations).
