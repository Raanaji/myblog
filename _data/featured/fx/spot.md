---
template: BlogPost
path: /spot
mockup: 
thumbnail:
date: 2019-07-21
name: FX Series - What is a Spot?
title: In our first post of FX Series we will discuss in brief about Foreign Exchange Spot Markets and Models
category: Quantitative Finance
description: 'Foreign Exchange Models in the FX Spot Market.'
tags:
  - FX 
  - Trading
  - Spot
  - Quantitative Finance
---

## Introduction to FX Spot Markets

A “spot” trade is an agreement to exchange some amount of one currency for another amount of another currency. Usually settles two business days after the trade date

Bilateral, over-the-counter transactions. Not exchange traded, or even traded on SEFs. Currency futures exist – we’ll discuss next week – but only a small part of the market

Not cleared Typically settled through CLS to manage settlement risk, only guarantees that you get back your side of the trade

## Spot Market Conventions

Currency: a currency, like EUR Currency pair: a pair of currencies, like EURUSD

The first currency in the pair is the “asset” currency, and the second is the “denominated” currency. Market chooses a convention for which way around to quote. “EURUSD” means “the price of a EUR in USD” (currently around 1.13 USD per EUR), “USDJPY” means “the price of a USD in JPY” (currently around 120 JPY per USD)

EURUSD, AUDUSD, NZDUSD, GBPUSD are quoted in USD per currency, USDJPY, USDCHF, and USDCAD are quoted in currency per USD. Normally convention is such that the price is >1.

![realizedvolatility](/assets/fxspot/realizedvolatility.png)

## Spot Market Dynamics

Spot is diffusive, occasional small jumps around economic releases etc, but only ~0.5% or less, once every month or two, Markets are never closed because of global trading. The realized volatility is normally in the 5-15% range, occasionally however it runs higher eg during the credit crisis. Some evidence for mean reversion in FX prices which has half life around 1 year which is also quite weak and is regardless not much of an effect on derivatives pricing methods.

## Currency Crosses

A “cross” is another term for a non-USD currency pair, eg EURJPY, which can replicate a currency cross spot trade with trades in the underlying USD pairs; eg buy 10.0M EUR, sell 10.9M CHF for 2 days in the future i.e.EURCHF spot price 1.0900.

Replication by buying 10.0M EUR, selling 11.3M USD; buying 11.3M USD, selling 10.9M CHF

EURUSD spot price 1.1300, USDCHF spot price 0.9646. The “triangle” is the spot market for the three currency pairs that are linked by this arbitrage

Tricky part comes when sometimes market-convention spot date is different for the three currency pairs in the triangle, sometimes due to “spot days” convention differences, however it is mostly due to holiday differences./

Spot date convention for crosses (assumes 2 business days). Move ahead one day, avoiding currency settlement holidays in each of the two non-USD currencies. Move ahead one more day, avoid currency settlement holidays in each of the two non-USD currencies **and** avoiding USD currency settlement holidays

Triangle arbitrage still holds when spot dates are different; it’s just that some of the trades are not exactly spot trades

They are forward trades to settlement dates that are not spot. When calculating the cross spot rate, need to make sure the USD forward rates are to the spot date of the currency cross

- F = forward FX rate to time T

- R = denominated currency interest rate

- Q = asset currency interest rate

- T = time to settlement (measured from spot date, T=0)

Currency cross spots often trade in their own markets, separate from the USD-pair markets but linked through the triangle arbitrage. Sometimes a cross is more liquid than a USD pair

EURCHF is more liquid than USDCHF in London hours. Generally executing the triangle arbitrage results in effective bid/ask spreads that are larger than the cross-specific market.

When making markets on currency crosses, traders need to look at the cross market but also at the triangle arbitrage to determine the best market to hedge their risk in