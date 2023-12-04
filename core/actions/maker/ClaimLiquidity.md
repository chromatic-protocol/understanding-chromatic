# ClaimLiquidity

```mermaid
sequenceDiagram
  autonumber
  participant R as Router
  box Market
  participant F as MarketLiquidityFacet
  end
  # participant V as Vault

  R->>+F: claimLiquidity
  F->>F: _claimLiquidity
  note right of F: transfer CLB token
  F->>R: claimLiquidityCallback
  note right of R: remove receipt 
```

## _claimLiquidity of MarketLiquidityFacet

```mermaid
sequenceDiagram
  autonumber
  participant F as MarketLiquidityFacet
  participant P as Pool
  
  box Bin
  participant B as LiquidityBin
  participant L as BinLiquidity
  end

  F->>P: acceptClaimLiquidity
  P->>B: acceptClaimLiquidity
  B-->>B: _settle
  B->>L: onClaimLiquidity
  L-->>L: update or delete _claimMintings
  L->>F: returns clbTokenAmount 
```


## settle of LiquidityBin

```mermaid
sequenceDiagram
  autonumber
  box LiquidityBin
    participant B as LiquidityBin
    participant C as BinClosedPosition
    participant P as BinPosition
    participant L as BinLiquidity
  end
  B->>C: settleClosingPosition
  B->>P: settlePendingPosition
  B->>L: settlePendingLiquidity

```

## settlePendingLiquidity of BinLiquidity

`_settlePending`에서 oracleVersion이 업데이트 되었다면, addLiquidity에서 요청한 deposit의 CLB amount를 확정하고 liquidity `total` 증가, `claimMintings` 업데이트

```mermaid
sequenceDiagram
  autonumber
  participant L as BinLiquidity
  participant T as clbToken
  participant V as Vault

  T->>L: totalSupply
  L->>L: _settlePending
  Note right of L: pendingDeposit, mintingAmount
  
  L->>L: _settleBurning
  Note right of L: burningAmount, pendingWithdrawal

  L->>T: mint or burn
  L->>V: onSettlePendingLiquidity
  Note right of V: update balance and pending states
```
