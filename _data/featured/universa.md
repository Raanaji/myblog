---
template: BlogPost
path: /universa
mockup: 
thumbnail:
date: 2020-08-01
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

> No matter the time period examined, Universa’s risk mitigation was decisively less costly and more effective All of this clearly demonstrates what makes a risk mitigation strategy effective: **how much protection is provided in bad markets versus how costly that protection is in good markets.** The typical mild diversifying strategies, such as in our conventional proxy portfolio, provide so little relative protection in bad markets. Large allocations to the strategy are required; because of that, the protection becomes an immense drag in good markets—what Mark Spitznagel calls “the plague of diworsification”. The totality of the payoff is what creates the portfolio effect.
>
> Ronald Lagnado, Ph.D.; Director - Research Universa Investments L.P.

For anyone who has read all of the memos published by Universa since 2016 knows what is at stake here. You (the investor) are insuring against the far left tail, not a 10% drawdown.  The only tail event (if you one can even call that so) was in August 2015 when the flash crash on August 24 happened. One only would have had made money on the OTM puts if  they were sold that day (and in the first hour of trading), and thus comes the point. As an equity investor one has to accept at least moderate vol to earn a return- it is the 20%, 30% and 40%+ drawdowns that tail risk hedging is intended to insure against.



![mitigation](/assets/universa/mitigation.png)

<figcaption>Comparing Risk Mitigation Strategies</figcaption>



That is up to the aggressiveness / risk tolerance of the individual investor. It's usually fairly obvious when volatility gets expensive. As Spitznagel explains in his book (using market making in the bond pits as an analogy for options), always take a one-tick loss, but how long you let your winners ride out is up to you.

> Now can we backtest these so called strategies? Well, if you backtested an OTM put writing strat in early 08' you would have found it very profitable, then in a few months you would have been in the bread line.

So is the tail hedging-black swan strategy equivalent to buying SPY Puts Deep OTM, (a.ka.a Nassim Taleb Strategy) ?

First of all to understand, this portfolio would spend a small percentage of its equity exposure every month buying 2-month put options that are about 30% out-of-the-money. After one month, those put options are sold and new ones bought according to the same methodology. The number of put options is determined such that the tail-hedged portfolio breaks even for a down 20% SPY fall.

This article below applies the strategy and does a stress test for a 20% downturn. [The Article](https://thefelderreport.com/2016/08/15/worried-about-a-stock-market-crash-heres-how-you-can-tail-hedge-your-portfolio/)

If you look at the inputs to the calculation, Everything makes sense, except for the way he forecasted Implied volatility-he added 10 points to the VIX. A similar strategy can be re-created with one small tweak - the put contracts (which we shall refer to as P ) were the only long term position in the account. Each position has its own hedge during trade duration ( this allows us to sleep easy at night and focus on work during the day) these positions are closed simultaneously when profit or loss is realized.

6 strike/exp P is chosen for the Year, 6 mo, and quarter respectively, then the capital is allocated to overlapping weekly and monthly options. This allows us to cover more time and different strikes / exp for the same capital outlay.

As the portfolio gains/loses value over the course of the quarter the P is adjusted accordingly toward its worthless expiration.

This money is considered **insurance** and is spent to expire worthless. When we had a market crash back in winter 3 years ago, one could have realized an entire years worth of gains in just a few days.

> Essentially what we have done in the past is to pick 6 price targets for the underlying. 4 targets (1 for each quarter) a 6mo and a 12 month target. Remember you are trying to hedge against a serious market drop so this is different than trying to chart moving averages etc.

We look at a few technical indicators tho I do not like technical trading, that mostly consider trend, volume, momentum and standard deviation.

**The following example demonstrates clearly why one does not need the horde of useless mathematical finance calculations to make it right in the market (just like we do not need to write a physics dissertation to learn how to cycle).**

**Example** ( with SPY ): Say SPY@280 would be - let’s say we are trading with a USD2500 account and want to hedge 10% of that. One may look 5 months out and see that a USD200 PUT is 4.10 which is more than your allocation. So instead we can  see that the May weeklies are 0.35. for USD35 we can pick up a single contract that expires in 40 - ish days. Greeks maths not withstanding the ATM USD280 puts are trading for like 6. So...... if we picked that USD200 PUT up for .35 and it decayed for a week or two in the event of a major drop in SPY we'd likely walk away with a few hundred dollars on that single contract. Probably a bit more than the **10% hedge**.

By picking several strikes and expirations throughout the year we are able to cover more area for a longer period of time with the defined risk of our hedge.

Managing these positions is interesting because sometimes there are spikes in volatility and what was 85% worthless a month ago now breaks even or even returns a few points. In those cases the position would be closed and rolled out and re-allocated if need be.

Ultimately we will need to decide what works best for your investment strategy. We could also buy more expensive closer to the money puts each day before market close and buy them back the next day. For most people however this doesn’t make much sense if we aren’t long any positions. it would be more about positioning ourselves to profit from a falling market which could be a strategy in itself for some. 🤷🏼‍♂️

> **REMEMBER THAT BEING IN A CASH POSITION IS IN ITSELF A HEDGE.** One can’t lose $ if one is not holding.

However, the above strategy has many assumptions and practically is not really a good way to do *HEDGE* per se. Essentially what we assumed is that the whole vol surface moves up by 10 points. In reality only our skew will matter, so the implied vol of 30% OTM strike may move by a different amount.

A better way to do it see how the implied vol of the 30% OTM option changed in that period.

Also there's the debate of whether to use an absolute vs relative vol shock.

Lets take the S&P 500. The implied volatility of the S&P 500 that we get from the prices of options, depends on the level of the S&P and the time to maturity. So we have 3 dimensions (volatility, price, time) here and we can build a surface.

The vol is measured at each price level and related to the time/strike at each node. It is a representation of how the vol of a certain price range compares to other options in the same chain. Basically, how cheap or rich an option is against its counterparts.

Overall, there really is not much insight one can gain from the memos of Universa or those strategies touted by Nassim Taleb. Most of those are mere marketing gimmicks, as we will see in our later posts. In truth, one should know when to not become a sucker, something even a street thug among the ranks of dixie mafia would have known.