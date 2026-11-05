---
name: hyperdrive-markets
description: Use this skill to integrate with the Hyperdrive markets, either through reading the market data, or interacting through lending and borrowing.
---

# Hyperdrive Market Integration

## Deployments

The deployed markets are available for llms at [https://docs.hyperdrive.finance/developers/contracts/llms.txt](https://docs.hyperdrive.finance/developers/contracts/llms.txt).

## Error Codes

When transacting against the market, it can throw errors. For market specific errors, these are thrown as an `error ErrorCode(uint256 code)` custom error where `code` is the ID of the specific error. The error codes are defined in the [error-codes.md](./references/error-codes.md) reference file.

## Market Components

Each Market is an isolated contract that is paired with an adapter, a risk model, and an optional hook. A Market is an ERC4626 vault for lenders whereby their deposits are lent out to borrowers and the interest accrues as yield via the standard ERC4626 accounting.

| Component  | Required | Description                                                                                                                                                  |
| ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Adapter    | Yes      | The adapter mediates the asynchronous conversion of loan assets to collateral (subscribe) and collateral back to loan assets (redeem) on behalf of a Market. |
| Risk Model | Yes      | The risk model supplies a Market with its borrow rate, LTV, liquidation discount, and liquidity fee parameters.                                              |
| Hook       | No       | The hook is an optional contract that can be used to add custom logic to the market and executed before and after public functions on the market.            |

## Loan Asset

The loan asset is the underlying ERC20 token that is lent out to borrowers and exists as the ERC4626 vault `asset()`.

## Collateral Asset

The collateral asset is the underlying ERC20 token that is used as collateral for loans. The collateral asset is defined as the `collateral()` function on the market.

## Virtual Shares

Deposits and borrows are represented as shares in the market. Every share to asset conversion pretends there are an extra 1e6 shares and 1 asset in the pool (totalShares + 1e6, totalAssets + 1). This sets the starting rate at 1e6 shares per asset and blocks first-depositor inflation and donation attacks on both the supply and debt share pools.

## Account Positions

An account can have multiple positions in a market, each with its own debt and collateral. The default position is always assumed to be 0.

### Health Score

Each position has a health score which ranges from 0-1000 where anything below 100 is considered liquidatable. The health score is calculated as follows:

```
min(1000, (collateral * redemption_rate * liquidation_ltv) * 100 / debt)
```

For any risk changes actions (ie, borrow, withdraw collateral) on a position, the health score is recalculated using the `ltv` instead of the `liquidation_ltv` and if it falls below 100, the action is denied.
