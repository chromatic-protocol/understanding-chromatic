# Chromatic Protocol Developer Documentation

## Introduction

Welcome to the Chromatic Protocol developer documentation. Chromatic Protocol is a decentralized perpetual future market protocol.

## Protocol Overview

### `Market`
- Opening a market is decenralized, allowing DAO to create a market.
- To prevent the proliferation of fragmented markets, a unique market is created for each `(oracle, settlement token)` pair.
- Participants:
  - `Taker`:
    - Engages in the market to predict market direction, initiating trades to gain profits or losses.
    - Pays a `trading fee` upon opening a position and an `interest fee` to the maker during the position's maintenance.
  - `Maker`:
    - Acts as the counterparty to the taker's directional trade, earning the opposite `PnL`.
    - Provides liquidity through a pool and earns passive income from taker's fees.

### `Underlying Asset`
- Can use any oracle's price as the underlying asset, allowing the creation of future prediction markets for various assets.

### `Settlement Token`
- Can use any settlement token approved by the protocol DAO for the market's payment currency.
- Separation of settlement tokens and underlying asset prices provides flexibility.

### `Trading Base Price`
- Lazy application of the next round's oracle price feed for order transactions.
- No separate futures price or basis, simplifying the design.

### `Taker`
- Position Opening:
  - Decides on the market direction (`long`/`short`), amount, and leverage.
  - Sets a `take-profit` and `loss-cut` for automatic exit conditions.
- Position Closing:
  - Can manually exit positions.
  - Automatic exits on reaching `take-profit` or `loss-cut`.

### `Maker`
- Supplies liquidity to takers and earns fees.
- Chooses their `trading fee rate` and deposits liquidity accordingly.
- Earns two types of risk-free income:
  - `Trading fee`: Collected from the taker upon position opening.
  - `Interest fee`: Paid by the taker during the position's maintenance.
- Receives a share of fees and interest from the each bins.

### `bins`
  - market consists of bins correspond to each trading fee rate


## overview of contracts

refer to [docs](https://chromatic.finance/docs/contracts/reference/overview).
