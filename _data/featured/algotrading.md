---
template: BlogPost
path: /algotrading
mockup: 
thumbnail:
date: 2010-10-23
name: A brief on Universa Investments trading strategies
title: What exactly does Universa Investment does which others fail to do?
category: Quantitative Finance
description: 'Strategy for securing left-tail of the portfolio is harsh but rewarding in the long run.'
tags:
  - universa 
  - left-tail
  - options
  - market crash
---

In the previous post on algorithms we discussed what type of search algos we use generally in financial engineering to help us sift through data. In this post we will truly discuss about trading assisted by algorithms.

## Introduction

The natural progression of a trading system is a fully automated platform for market making and trading on various securities, known as an Algo trading system Trading systems began in the 1980s and 1990s as a platform for trade booking, position keeping, risk management, and pricing

In the 2000s we added electronic trading capability to a trading platform with simple automation. Algo trading became hugely popular starting around 10 years ago and around 2010 became a mainstay of any trading system

Algo trading typically encompasses order execution (i.e. aggressing the market) and market making to profit off the bid/offer spread (i.e. providing liquidity) – also RFQ pricing and Agency. Being able to provide the strategies with appropriate inputs and outputs is how you set up the strategy – that’s half the battle. The rest is being able to write a profitable strategy that can make sense of the inputs and leverage the outputs to make a profit

## Regulating e-Trading

The 1987 stock market crash (Black Monday) spiraled into a full-blown crash because there were no buyers able to take the other side of the market in a selloff – brokers refused to pick up the phone. Regulation necessarily ensued to ensure that electronic platforms would promote liquidity in markets where traditional brokers vanished – as a result, the Small Order Execution System (SOES) was born to match buyers and sellers

Various electronic market makers emerged trying to game the SOES platform with computer algorithms to profit from slow manual traders (i.e. the first high frequency traders). Decimalization in 2001 on the Nasdaq served to tighten bid/offer spreads, thus beginning to squeeze out the traditional specialists and traders and giving rise to the explosion of electronic trading

Electronic markets became fragmented as various exchanges and dark pools were born. Regulation NMS in 2005 served to protect the retail investor by mandating that orders be routed to the exchange with the best price

Reg NMS ended up harming the investor as the best price (particularly where the size is so small) is not necessarily best for the investor to fill the entire order – high frequency traders sniff out a small part of the order and front-run back to other markets

The financial crisis of 2008 served to create enough volatility for high frequency trading strategies to become immensely profitable. Volatility creates price movement where speed becomes crucial – more chance for a high frequency trader to be able to use the information of a price movement to trade against less sophisticated market participants

USD21 billion in profits for HFTs in 2008 – profits have more recently plummeted as volatility subsided in the wake of the financial crisis (USD1.3 billion in 2014). Flash orders were originally used to sniff out where large institutional investors were trading – now illegal!

What began as a revolution in Equities trading has now spread to FX and Rates (Currencies are heavily traded electronically, with Treasuries following suit)

## Dodd Frank and Swaps Electronic Trading

Dodd-Frank regulation in 2010 has mandated that all Swaps (Equity Swaps, Interest Rate Swaps, and Credit Default Swaps) be traded electronically below a certain block size

Intention to ensure more transparency in these traditionally opaque markets to avoid potential for large counterparty risk and catastrophic loss. Swap Execution Facilities (SEFs) have risen to promote electronic trading of Swaps

Since 2013, there has been an intense focus around the Algo trading of Swaps (Citadel launched a broker-dealer with the intention to trade Swaps electronically)

## Electronic Trading Boom

Over 80% of all Equity market trades done by a computer-based algo, and over 60% of all US Treasury trading is done by a computer-based algorithm.

Algo trading is on the increase as we see a boom in electronic trading in all markets – first Equities, then FX, then Rates. Regulation is being ramped up in response to the volatility that Algo trading potentially creates, highlighted by the following two examples:

The Flash Crash of 2010 where it is believed that market manipulation from one trader led to the huge swings in the stock market. Large price movements in the 10Y and 30Y sectors of US Treasuries in 2014 where Algo trading accounted for over 90% of all trading in a 12-minute window that caused yields to plummet more than on the day of the Lehman bankruptcy

Compliance departments at banks and hedge funds are being revamped to tackle the onslaught of regulation – lobbying groups too!

## Electronic Trading in a CLOB

Most electronic trading across all products occurs in a Central Limit Order Book (a CLOB). Buyers and sellers are matched in a CLOB. Liquidity providers place two-way resting orders in the CLOB (i.e. those that are not cancelled right away) – profit off the bid/offer spread or liquidity provider rebate

Liquidity takers aggress the market – if they cross the bid/offer spread then a trade happens (liquidity takers pay a fee to the exchange, part of which goes to the market and part of which forms the rebate to the liquidity provider on the other side of the trade)

- Refresher – we have the following types of CLOB orders (in a simplistic model):

- FOK – fill-or-kill, where we either fill the entire order at the given price or the order is killed

- IOC – immediate-or-cancel, where we can fill the order partially for available liquidity and then cancel the remainder immediately

- Market – where we do not specify a price and simply execute at the current market price for the prevailing quantity

- Limit – where we place a resting order that gets filled once the price hits our limit (or if it is better)

- Stop – where we place an order (in conjunction with a limit order) which executes when the price gets only as bad (or worse) than our price

## Electronic Trading as an RFQ Protocol

An RFQ (request-for-quote) occurs in most fixed income markets only. Also known as B2C (business-to-customer) trading. A client on the buy-side sends an RFQ to multiple dealers, who then quote back a price to the client – the client trades on the best price generally. A very manual process on the buy-side (the sell-side will generally automate the response via an algo)

## Ingredients to Algo Trading

Four forms of Algo trading:

- Liquidity taking algos
- Liquidity providing algos
- RFQ pricing algos
- Agency algos

Liquidity taking algos consist of the following workflow:

- Market Data inputs to inform a trading decision in the algo
- Algo strategy logic to make the trading decision
- Order execution connectivity to place an order as a result of the algo decision

Liquidity providing algos consist of the following workflow:

- Market Data inputs to inform a pricing decision in the algo
- Algo strategy logic to influence the market making decision
- Order execution connectivity to place a resting order (new order or update) as a result of the algo decision

RFQ pricing algos consist of the following workflow:

- Market Data inputs to inform a trading decision in the algo 
- Algo strategy logic to make the trading decision
- RFQ quote to be sent back to the client

Agency algos consist of the following workflow:

- Client wishes to work a large order
- Agency algo splits the order into smaller child orders
- Agency algo works the child orders to achieve some target overall price
- Agency algo efficiently works the child orders on multiple markets

Tick-to-order or tick-to-quote latency is vital in the performance of the algo strategy – often look to optimize to microsecond latency (or potentially nanoseconds)

## Latency Arbitrage

Speed has become crucial in today’s world of electronic trading. Latency arbitrage example:

- An institutional investor is trading both Cash (on ICap) and Futures (on the CME)
- If these orders are sent out at the same time, high frequency traders may see an order on one market and then race to front-run the order at the other market
- This is known as the latency arbitrage

Sophisticated market participants develop the fastest electronic trading and algo technology to profit from latency arbitrage. They are co-located at the various electronic markets for fastest access to market data and fastest speed of order execution. They leverage the fastest WAN connections to the exchange (fiber optic cable, microwave, etc)

Smart Order Routers have been developed to counteract the latency arbitrage by measuring network latency to various electronic markets and time when we send the order to ensure they reach the markets at the same time (within an acceptable tolerance)

## Connectivity

The fundamental building block of an algo trading platform is the connectivity. This provides the input into the Algo strategies themselves. Connectivity needs to be as fast as possible.

Slow connectivity means the Algos are reacting to potentially stale data, meaning the decision making could produce invalid results that result in no actual trading, or worse yet, unwanted exposure to stale prices when providing liquidity resulting in loss

The connectivity essentially provides the inputs and outputs to the Algos. Inputs typically thus consist of market data – the most common form of market data is the full order book from an exchange

Most Algo strategies will react to updates in the order book and make a decision based off some signal (either to aggress the market or provide liquidity). The exact logic of what generates the decision to provide or take liquidity is encapsulated in the strategy code itself (which we go into detail in the next section)