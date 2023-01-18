---
title: Borrowing Protocols
date: '2021-05-02'
tags: ['crypto', 'defi']
draft: false
summary: 'A short dive into borrowing protocols'
---

# Asset Price Movements and Liquidations

Synthetic Asset protocols ie DAI, Mirror, Synthetix, Yeti

- Needs oracle that provides prices of collateral asset
- Can mint new DAI when `O_t * a * c >= b` where O_t is the oracle price, a is the amount of collateral, c is the collateral factor(usually less than 1), b is the amount of synthetic tokens
- If the oracle updates and cause the above invariant not to hold, a liquidation of the position is allowed(This is very bad for the person who took out their position!!)

For the multi-collateral case, O_t, a, c, b are all vectors and the following invariant is used
`sum over collaterals(O_t * a * c) >= b`

If you have a mixture of two assets(ie 50% eth and 50% bitcoin), as long as the oracle prices are not perfectly correlated, there is a smaller chance that the collaterals move in a way to get you liquidated.

### Arguments for not doing this?

Might be hard to set the parameters for `O_t, a, c, b`, but gauntlet already does this for collateral ratios on compound and aave

### Why is everything correlated?

Things are empirically correlated and the why doesn't really matter.

People who invest in an asset class want to be diversified, if the bitcoin in their portfolio goes up, they want to rebalance to stay diversified so they sell part of their bitcoin and buy eth, causing the eth price to go up.

# Risk Modeling for Protocols with Liquidations

## Example

Max Loan Size
Supply vector x collateral ratio \* prices
Asset | Collateral factor
ETH | .825
DAI | .825
WBTC | .7

# Mechanics of Liquidation

When liquidation is allowed, liquidators repay the borrowed amount and gets a portion of the collateral + some bonus. THe liquidator immediately market dumps the collateral + bonus.

There's a routing library in Julia that Bain cap wrote(doesn't support curve/univ3)

## Cascading Liquidation Feedback Loop

Liquidations -> Market Dump -> Price Drop -> More Liquidation

Algorithm:

1. Input current state of lending markets, exchanges, prices
   While True:
   Check for liquidations
   If liquidations:
   Route them over exchange graph, update exchange states, update prices to post dump prices

Why?
Allows us to calculate structural component expected loss vector and variance covariance matrix based on actual state of the chain
Allows us to better understand price trajectories. Can sell to people who care

### Questions

What's the intuition behind multi-collateral synthetics being less likely to liquidate? What about non-stablecoins?
