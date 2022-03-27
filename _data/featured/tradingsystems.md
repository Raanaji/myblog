---
template: BlogPost
path: /tradingsystems
mockup: 
thumbnail:
date: 2019-11-13
name: The basics of trading systems
title: We will discuss in brief about trading systems from an institutional perspective 
category: Quantitative Finance
description: 'Institutional trading systems explained.'
tags:
  - systems 
  - quantitative finance
  - trading
---

In the previous post on algorithms we discussed what type of search algos we use generally in financial engineering to help us sift through data. In this post we will truly discuss about trading assisted by algorithms.

## Introduction

We now introduce trading systems in finance, focusing on a bond trading system as our example

We look at the key ingredients to a trading system in an SOA (Service Oriented Architecture) platform:

- Product Service
- Trade Booking Service
- Position Service
- Risk Service
- Pricing Service
- Execution Service
- Market Data Service
- StreamingService
- InquiryService
- Historical Data Service

These services co-exist in an SOA trading system and can be broken up into separate components or all housed in one process. We describe all these services and then discuss how we may distribute these services

## Service with Listeners

Let’s evolve our notion of a Service with listeners as follows

```c++
template<typename V>
class ServiceListener
{
  virtual void ProcessAdd(V &data) = 0;
  virtual void ProcessRemove(V &data) = 0;
  virtual void ProcessUpdate(V &data) = 0;
};
template<typename K, typename V>
class Service
{
public:
  virtual V& GetData(K key) = 0;
  virtual void AddListener(const ServiceListener<V> &listener) = 0;
  virtual list< ServiceListener<V>* > GetListeners() = 0;
};
```

Here we define the Service with the ability to add listeners (and retrieve them). Listeners can be notified for addition, removal, and updates of data. Thus, in this way, we can define interested listeners that can handle various events

This forms a crucial element of Service Oriented Architecture where we can register listeners to react to events on the data in a service. Events can be pushed to middleware (by a listener even – say a `DataPublisherListener`)

## Product Service

We have defined a ProductService in the previous class. This serves as a crucial building block in our trading system to provide product reference data throughout our platform

E.g. for a bond trading system, all supported bond securities should be defined in our ProductService and reference data made available to any interested subscribers to the service

## Trade Booking Service

When we trade in the system, we need to be able to book trades into a central repository (i.e. a trade database). Such trade booking is done via a trade booking system

STP (straight through processing) is functionality that connects to markets for a post-trade feed and automatically books trades into our trading platform via our TradeBookingService Attributes of a trade are:

- Product
- Trade Id
- Book
- Quantity in Dollar notional amount (or any currency)
- Side (Buy or Sell)

Operations on our TradeBookingService are:

- Book a trade (which can flow into our position service discussed in the next slide)

## Position Service

Once a trade is booked, it generates a position (a long position if the trade is a Buy, a short position if the trade is a Sell) – multiple trades are netted out to produce a position.

We need to know our position at any given point in time in a particular product. Understanding our positions across many different products gives us a holistic view of our risk and exposure that is vital to trading

Our position gives us a view on balance sheet utilization (position is netted, balance sheet is gross). A position service consumes trades (in a given product and book) and produces a net position for that security that can be queried by consumers of the PositionService. Typically a `TradeBookingService` invokes the PositionService with a trade once a trade is done

Attributes of a position are: Product

- Dollar position amount (positive is long, negative is short) for a given book
- An aggregate position across all books for the product (netting out longs and shorts)

Operations on our PositionService are:

- Add trade (which generates a resulting position (incorporated into the existing position potentially)



## Risk Service

In order for our traders (or any automated trading components) to be able to make intelligent trading decisions, we need to be able to calculate risk appropriately

A RiskService is able to calculate risk on fixed income securities. We use PV01 risk metrics in the fixed income world to calculate risk (i.e. using the equivalent price movement in a basis point yield movement to calculate an overall dollar movement for a trade or position)

We calculate bucketed risk across a group of securities. **Example of bucketed risk** – we calculate aggregate risk across all securities (netted out) in each of the following buckets:

- Front-End: 2Y and 3Y sectors
- Belly: 5Y, 7Y, and 10Y sectors
- Long-End: 30Y sector

Attributes of PV01 risk are:

- Product
- PV01 value
- Quantity that the PV01 value applies to

Operations on our RiskService are:

- Add a position to update our risk
- Get bucketed risk across a bucket sector of products



## Use of Position/Risk Services

We can split trading into three forms, with each needing a consistent view of positions and risk:

Proprietary trading means that we take positions (long or short) in order to take a view on where the market is going for a particular security, sector, or a whole asset class – primarily the job of hedge funds, mutual funds, pension funds, etc for various purposes (not a bank or broker/dealer).

Principal trading means we take risk as a dealer to be able to make markets to our clients, i.e. we trade on information where we think (or know) the client is trading to be able to provide liquidity back to our clients – which means we take positions in advance of trading with our clients.

Agency trading means we trade on behalf of the client and take no risk or positions (instead simply charging a commission on each trade) – primarily the job of banks and broker/dealers.

In order to effectively trade in any of these ways, we need a consistent view of our positions across many different products and overall risk exposure (bucketed, etc).

## Auto-Hedging and the Risk Service

An Auto-Hedger is able to automatically hedge our risk. We can hedge trade-by-trade (often expensive) or based on bucketed risk exceeding a certain threshold. Complex algorithms around determining how and when we place our hedges – this is proprietary information!

An Auto-Hedger needs a RiskService to know exactly what our current risk is and to be notified on risk updates to be able to hedge appropriately via its algorithm.

## Pricing Service

A pricing service generates internal prices for our own valuation of a universe of products (i.e. a fair value mid and a market bid/offer spread around it).

A pricing service is critical to any form of trading to inform the trader or automated trading platform of where we think the market is for a given security/ Low latency can often be critical to any pricing service. 

A pricing service can conflate updates on a given security to ensure that only the latest tick arrives to the consumer (if the feed is ticking very fast – perhaps too fast for some consumers, e.g. a GUI, to handle). Use of optimal data structures is critical for a pricing service. A pricing service can sometimes be separated into mid price generation (for fair value mid, closing marks, risk metrics, electronic trading input, etc), Bid/offer generation for actual trading. It is generally better to keep all price generation in one place for optimal performance

Attributes of a price are:

- Product

- Mid price

- Bid/offer spread

Operations on our PricingService are:

- Specific to each product (e.g. can provide driving relationships for bonds, etc)

## Pricing Service in Practice

For our three forms of trading, our pricing service is crucial:

- Proprietary trading – in order to trade on any security for taking a view on the market, we need to know the current market value of the security we are trading

- Principal trading – in order to provide liquidity back to our clients, we need to know where the current market is (and profit from the bid/offer spread)

- Agency trading – in order to trade on behalf of a client on an exchange, we need to know where the market is trading to ensure we get our client the best price

## US Treasuries Pricing Service

A pricing service can be used to generate prices for US Treasury securities. US Treasury pricing for On-The-Run securities (OTRs) generally uses the order book to generate a mid (volume weighted, average best, etc) – with controls around where the market has last traded

A bid/offer around the mid is generated using the entire order book with some algorithm with the order book as an input. Off-The-Run Treasuries are generally driven from OTR Treasuries. Most simple form is OTR directly driving an Off-The-Run

Often we need a combination of OTRs as the securities drift further away from a particular OTR (i.e. as OTRs roll and the Off-The-Run becomes older in relation). Bid/offers can be further widened using various controls based on security, client, and quantity

Further widening or tightening of the bid/offer spread is done manually or by algorithm (e.g. based on a market event, liquidity conditions, etc)

## Execution Service

An execution service is used to execute orders on a given market. We have the following types of orders (in a simplistic model). FOK – fill-or-kill, where we either fill the entire order at the given price or the order is killed

IOC – immediate-or-cancel, where we can fill the order partially for available liquidity and then cancel the remainder immediately. Market – where we do not specify a price and simply execute at the current market price for the prevailing quantity

Limit – where we place a resting order that gets filled once the price hits our limit (or if it is better). Stop – where we place an order (in conjunction with a limit order) which executes when the price gets only as bad (or worse) than our price We can divide our order into several child orders at smaller sizes (so as not to move the market in one shot) – each refers back to the parent order via a parent order id

We can place visible quantity (that market participants can see) or hidden quantity (for an iceberg order)

Attributes of an order are:

- Product
- Order Id
- Order Type
- Price
- Visible Quantity
- Hidden Quantity
- Parent Order Id
- Is Child Order

Operations on our ExecutionService are:

- Execute order on a particular market

A market data service provides market data to interested subscribers

Essentially provides a view on the complete order book for a given market (or markets)

The market data service can aggregate liquidity at all price points

The service can provide the best bid/offer in the market (for one or more markets)

Low latency is CRUCIAL in the development of a market data service

Data structures need to be highly optimized!

Zero garbage collection important for Java – effective use of memory important for C++

Updates can again be conflated for slower consumers to process (i.e. we don’t necessarily need all updates to the order book – just the most recent one)

Algo trading generally needs the entire view of the order book (with all orders non-aggregated) as fast as possible – ITCH market data feed an industry standard for market data connectivity in the algo trading spaceMarket Data Service

Attributes of market data are:

Product

A bid stack and offer stack (with each entry in the stack constituting a price and quantity with a level)

Operations on our MarketDataService are:

Retrieve the best bid/offer for a particular product

Aggregate market data at all price points to create a new bid/offer stack

## Streaming Service

A streaming service is where we provide liquidity on markets (either firm or indicative)

This is a basic form of market making, i.e. we provide liquidity to electronic markets by streaming our prices to the market (and hence take principal risk as a result)

We can stream prices to markets as limit orders (for a central limit order book exchange) or just as an open one-way stream (depending on the market)

Attributes of a price stream are:

- Product

- Bid price

- Offer price

- Bid visible quantity

- Offer visible quantity

- Bid hidden quantity

- Offer hidden quantity

Operations on our StreamingService are:

- Publish price to the market

## Algo Trading and the Execution/MarketData/Streaming Services

Algo trading generally leverages a MarketDataService for viewing the entire order book. The algos will use current market data to be able to generate trading signals based on some algorithm. These trading signals get translated into orders placed on an exchange

Either an ExecutionService is used by the algo to place orders, or a market making algo will provide liquidity by streaming out prices in the Streaming Service.

## Inquiry Service

An inquiry is a request for quote from a client, also known as an RFQ. An inquiry is a way for a client to send a request to trade to multiple dealers. It is a way for dealers to provide liquidity on demand to clients rather than needing to stream out prices to a client

Attributes of an inquiry are:

- Inquiry Id

- Product
- Side
- Quantity
- Price (as specified by the dealer)
- State (for the lifecycle of an inquiry)

Operations on our InquiryService are:

- Send a quote back to the client
- Reject the inquiry

## Historical Data Service

We persist historical data for historical analysis (e.g. to run simulations against the data, back test our strategies, etc). Persisting to KDB is a popular choice. Historical data has no attribute as this is any of the types described in this class (market data, inquiry, price stream, product, order, etc)

Operations on our HistoricalDataService are: Persist data.

## Trading Systems are SOA!

SOA is a crucial element to developing trading systems in the financial world these days

Develop independent components that can co-exist with each other and be distributed in the platform

Culmination of all the concepts we have been discussing in the financial computing – data structures, algorithms, distributed systems, service oriented architecture!

As a quant, it is highly likely that you will either develop strategies in a trading system or consume data on the edges of the platform (or publish data to it)

Next post will be… to develop a bond trading system, perhaps.