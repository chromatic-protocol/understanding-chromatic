# Simple Reference LP (liquidity provider)

## Overview
A single Liquidity Provider (LP) is a vehicle that provides and manages liquidity across multiple bins of a target market, implementing the maker's strategy.
- In other words, it provides a strategy for supplying liquidity to multiple liquidity bins of a market.
- It executes a strategy of adjusting fund allocation ratios through `rebalancing` by the keeper.
- The target of the LP's maker strategy is a single market, meaning
  - LP:market = 1:1
  - market:LP = 1:N
- Minting/burning of ERC20 tokens is executed when providing/retrieving liquidity to/from the LP.

## Operational Mechanism
- Maintain the target liquidity supply ratio.
  - `utilizationTargetBPS`: `0 ~ 10000`
  - Immutable.
- Distribution ratios for each bin.
  - `distributionRates`: `sum(distributionRates) = totalRate`
  - Up to 72 distribution ratios for each bin.
  - Ratios will change due to PNL, and no rebalancing is done to match these ratios.
- Periodically check rebalancing using the keeper based on rebalancing conditions.
  - `rebalanceCheckingInterval`: Rebalancing execution interval.
    - Immutable.

## overview

### creating

```mermaid
sequenceDiagram
  autonumber
  participant V as ChromaticLP
  participant M as ChromaticMarket
  participant O as Oracle
  participant T as TaskRegistry
  participant K as Keeper
  V ->> T: createTask(rebalance)
  V -> K: after deploy
  loop Every `rebalanceCheckingInterval`
    K -->> V: resolveRebalance
    opt resolved
      K ->> V: rebalance
    end
  end
```

- Register the `rebalance` task to be executed at the configured intervals (`t`) upon LP creation in the task registry of Automation framework (for example Gelato).


### ChromaticLPRegistry 
- To supply liquidity to a market, users should be able to query LPs dedicated to liquidity provision for specific markets.


### ChromaticLP

```mermaid
stateDiagram-v2
  state ChromaticLP {
    state Tokens {
      SettlementToken
      CLBs
    }
    state Receipts {
    AddReceipts
    RemoveReceipts
    }
  }
  SettlementToken --> AddReceipts: addLiquidity/ rebalance
  AddReceipts --> CLBs: settle
  CLBs --> RemoveReceipts: removeLiquidity / rebalance
  RemoveReceipts --> SettlementToken: settle
```


#### addLiquidity

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant V as ChromaticLP
  participant M as ChromaticMarket
  participant O as Oracle
  participant K as Keeper

  U ->>+V: addLiquidity
  V ->> V: _settle()
  V ->>M: addLiquidityBatch
  M ->>V: addLiquidityBatchCallback
  V ->>M: transfer settlementToken
  M -->>V: returns receipts
  V -->>V: store receipts
  V ->> K: createTask(settle)

  V -->>-U: returns LP receipts
  loop check oracle updated
    K -->>V: resolveTask(settle)
  end
  O -->>O: sync()
  U -->K: nextRound
  K -->>V: resolveTask(settle)
  V --> K: returns resolved

  K ->> +V: settle() / _settleAddLiquidity
  V ->> M: claimLiquidityBatch
  M ->> V: claimLiquidityBatchCallback
  M -->> M: remove receipt
  M -->> V: transfer CLBs
  V ->>-U: tranfer LP Token
  V ->>K: cancelTask(settle)
```

#### removeLiquidity

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant V as ChromaticLP
  participant M as ChromaticMarket
  participant O as Oracle
  participant K as Keeper

  U ->>+V: removeLiquidity
  V ->> V: _settle()
  V ->>M: removeLiquidityBatch
  M ->>V: removeLiquidityBatchCallback
  V ->>M: transfer CLB tokens
  M -->>V: returns receipts
  V --> V: store receipts
  V ->> K: createTask(settle)
  V -->>-U: returns LP receipt
  loop check oracle updated
    K -->>V: resolveTask(settle)
  end
  O -->>O: sync()
  U -->K: nextRound
  K -->>V: resolveTask(settle)
  V --> K: returns resolved
  K ->> +V: settle() / _settleRemoveLiquidity
  V ->> M: withdrawLiquidityBatch
  M ->> V: withdrawLiquidityBatchCallback
  M -->> M: remove receipt
  M -->> V: transfer settlement Token
  V ->>-U: tranfer settlement Token
  V ->>K: cancelTask(settle)
```


#### rebalancing

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant V as ChromaticLP
  participant M as Market
  participant O as Oracle
  participant T as TaskRegistry
  participant K as Keeper

  K ->>V: rebalance()

  opt settle
    V ->> +M: claimLiquidityBatch / withdrawLiquidityBatch
    M -->> -V: transfer CLBs / settlement token
  end

  alt reserved > `k` + `p`
    V->>M: addLiquidityBatch
    M ->>V: addLiquidityBatchCallback
    V ->>M: transfer settlementToken
    M -->>V: returns receipts
    V -->>V: store receipts
    V ->>T: createTask(settle)
  else reserved < `k` - `p`
    V ->>M: removeLiquidityBatch
    M ->>V: removeLiquidityBatchCallback
    V ->>M: transfer clb tokens
    M -->>V: returns receipts
    V --> V: store receipts
    V ->>T: createTask(settle)
  end
```


#### settle

```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant V as ChromaticLP
  participant M as ChromaticMarket
  participant O as Oracle
  participant T as TaskRegistry
  participant K as Keeper

  K ->> V: settle
  opt pending RemoveReceits exist
    V ->> +M: claimLiquidityBatch
    M -->> -V: transfer CLBs
    opt User's receipt of AddLiquidity
      V ->> U: transfer LP Token
    end
  end
  opt pending AddReceits exist
    V ->> +M: withdrawLiquidityBatch
    M -->> -V: transfer settlement token
    opt User's receipt of RemoveLiquidity
      V ->> U: transfer settlementToken
    end
  end
  V ->> T: cancleTask(settle)
```
