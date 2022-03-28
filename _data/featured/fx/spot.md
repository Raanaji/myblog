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