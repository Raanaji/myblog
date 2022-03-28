---
template: BlogPost
path: /toyalgo
mockup: 
thumbnail:
date: 2019-07-31
name: FX Series - Simulating a simple algorithm
title: In our this post we will implement a variation of the toy simulation algorithm
category: Quantitative Finance
description: 'Simulating a Simple Foreign Exchange Algorithms.'
tags:
  - FX 
  - Trading
  - Simulation
  - Quantitative Finance
---

## Assumptions

Model parameters to assume:

1. Spot starts at 1
2. Volatility is 10%/year
3. Poisson frequency $\lambda$ for client trade arrival is 1 trade/second
4. Each client trade that happens delivers a position of either +1 unit of the asset or -1 unit of the asset, with even odds
5. Bid/ask spread for client trades is 1bp
    ◦ Receive PNL equal to 1bp*spot*50% on each client trade (since client trades always have unit notional in this simulation)
6. Bid/ask spread for inter-dealer hedge trades is 2bp
   ◦ Pay PNL equal to 2bp*spot*hedge notional*50% on each hedge trade
7. A delta limit of 3 units before the algorithm executes a hedge in the inter-dealer market. 

We use a time step Δ𝑡Δt equal to 0.1/𝜆 , we assume that only a single client trade can happen in each time step (with probability equal to 1−𝑒^{−𝜆Δ𝑡}). We use 500 time steps and a number of simulation runs to give sufficient convergence.

When converted between seconds and years, we assume 260 (trading) days per year.

The variation in the algorithm: when the algorithm decides to hedge, it can do a partial hedge, where it trades such that the net risk is equal to the delta limit (either positive or negative depending on whether the original position was above the delta limit or below -1*delta limit); or it can do a full hedge, like in the algorithm we discussed in class, where the net risk is reduced to zero.

We then use the Sharpe ratio of the simulation PNL distribution to determine which of those two hedging approaches is better.

```python
import numpy as np
import scipy
import math
```

```python
class ElectronicHedgingEngine:
    def __init__(self, spot, vol, lamb, clientS, dealerS):
        '''
        spot: initial spot price
        vol: volatility per year
        lamb: client trade arrival follows Possion process, lambda of this Possion process
        clientS: Bid/ask spread for client trades
        dealerS: Bid/ask spread for inter-dealer hedge trades
        '''
        self.spot = spot
        self.vol = vol
        self.lamb = lamb
        self.clientSpread = clientS
        self.dealerSpread = dealerS

    def sim(self, dt, steps, N, deltaLimit, partialHedge = False):
        '''
        dt: time step
        steps: number of time steps
        N: number of simulation
        deltaLimit: delta limit
        partialHedge: if false, hedge to zero delta, if true, hedge to delta limit.
        '''
        
        # initialize all simulations' variable as numpy array
        spots = np.repeat(self.spot, N)
        deltas = np.zeros(N)
        pnls = np.zeros(N)
        
        prob = 1 - np.exp(-self.lamb*dt)
        
        for i in range(steps):         
            # accept-rejection, if client send trade
            AR = scipy.random.uniform(0, 1, N)
            tradeHappen = scipy.less(AR, prob).astype(int)
            
            # trade direction, +1 buy, -1 sell
            tradeDirection = scipy.random.binomial(1, 0.5, N) * 2 - 1
          
            # change delta and pnl  
            deltas += tradeHappen*tradeDirection
            pnls += tradeHappen * self.clientSpread * spots / 2.0
                   
            # if delta greater than upper bound, hedge
            upperExceed = scipy.greater(deltas, deltaLimit).astype(int)           
            if partialHedge is False:
                pnls -= deltas * upperExceed * self.dealerSpread * spots / 2.0
                deltas *= scipy.logical_not(upperExceed)
            else:
                pnls -= (deltas - deltaLimit) * upperExceed * self.dealerSpread * spots / 2.0
                deltas = deltas * scipy.logical_not(upperExceed) + deltaLimit * upperExceed

            # if delta smaller than lower bound, hedge
            lowerExceed = scipy.less(deltas, -deltaLimit).astype(int)           
            if partialHedge is False:
                pnls += deltas * lowerExceed * self.dealerSpread * spots / 2.0
                deltas *= scipy.logical_not(lowerExceed)
            else:
                pnls += (deltas + deltaLimit) * lowerExceed * self.dealerSpread * spots / 2.0
                deltas = deltas * scipy.logical_not(lowerExceed) - deltaLimit * lowerExceed
        
            
            # update spot
            dW = scipy.random.normal(0, np.sqrt(dt), N)
            dS = self.vol * spots * dW
            spots += dS
            
            # update pnl
            pnls += deltas * dS
        
        # return Sharpe Ratio
        return  pnls.mean()/pnls.std()
```

```python
# parameters
spot = 1.0
vol = 0.1/math.sqrt(260.0) # 260 days per year
lamb = 1.0*60*60*24 # seconds per day
clientSpread = 1e-4
dealerSpread = 2e-4
dt = 0.1/lam
steps = 500

# Create simulator
engine = ElectronicHedgingEngine(spot,vol,lamb, clientSpread,dealerSpread)
```

```python
print("Fully hedge strategy:")
for N in [1e2,1e3,1e4,1e5,1e6]:
    res = engine.sim(dt,steps,int(N),3.0,False)
    print("run ", N, "times: Sharpe Ratio is ", res)
```

```
#OUTPUT
Fully hedge strategy:
run  100.0 times: Sharpe Ratio is  1.8864797354854153
run  1000.0 times: Sharpe Ratio is  2.0581818346642886
run  10000.0 times: Sharpe Ratio is  2.097749039029468
run  100000.0 times: Sharpe Ratio is  2.070983848649503
run  1000000.0 times: Sharpe Ratio is  2.0629030234765793
```

```python
print("Partial hedge strategy:")
for N in [1e2,1e3,1e4,1e5,1e6]:
    res = engine.sim(dt,steps,int(N),3.0,True)
    print("run ", N, "times: Sharpe Ratio is ", res)
```

```
#OUTPUT
Partial hedge strategy:
run  100.0 times: Sharpe Ratio is  2.971481719687007
run  1000.0 times: Sharpe Ratio is  3.3916633352494143
run  10000.0 times: Sharpe Ratio is  3.352140100811335
run  100000.0 times: Sharpe Ratio is  3.347430240262172
run  1000000.0 times: Sharpe Ratio is  3.3347363476561105
```

## Interpretation

As we can see from above results, both strategies converge when number of simulation is a million. (variation smaller than 1%)

We can see partial hedge strategy has a higher Sharpe ratio. The intuition is, partial hedge strategy takes more risk, so it should be award by market. Thus partial hedge strategy has higher Sharpe ratio.

```python
"""
Copyright: Copyright (C) 2015  Washington Square Technologies - All Rights Reserved
Description: Functions for running a simulation of electronic trading. Model is:

                * Spot follows a geometric Brownian motion with zero drift
                * Client trades happen according to a Poisson process
                * Client trades pay a fee of half the client bid/ask spread, defined
                  as a fraction of current spot
                * Client trades are always of unit notional, with even odds of delivering
                  a long or short position
                * Hedging happens when the net position reaches a delta threshold. Two
                  types of hedging supported: one where it rebalances the position to
                  zero whenever it trades; and one where it rebalances the position to
                  the delta threshold.
"""

from   math import exp, sqrt
import scipy


def run_simulation_fast(vol, lam, sprd_client, sprd_dealer, delta_lim, hedge_style, dt, nsteps, nruns, seed):
    '''Runs a Monte Carlo simulation and returns statics on PNL, client trades, and hedge trades.
    "_fast" because it uses vectorized operations.
    
    vol:         lognormal volatility of the spot process
    lam:         Poisson process frequency
    sprd_client: fractional bid/ask spread for client trades. eg 1e-4 means 1bp.
    sprd_dealer: fractional bid/ask spread for inter-dealer hedge trades. eg 1e-4 means 1bp.
    delta_lim:   the delta limit at or beyond which the machine will hedge in the inter-dealer market
    hedge_style: 'Zero' or 'Edge', defining the hedging style. 'Zero' means hedge to zero position,
                 'Edge' means hedge to the nearer delta limit.
    dt:          length of a time step
    nsteps:      number of time steps for each run of the simulation
    nruns:       number of Monte Carlo runs
    seed:        RNG seed
    '''
    
    scipy.random.seed(seed)
    
    trade_prob = 1 - exp(-lam*dt)
    sqrtdt     = sqrt(dt)
    
    spots  = scipy.zeros(nruns) + 1 # initial spot == 1
    posns  = scipy.zeros(nruns)
    trades = scipy.zeros(nruns)
    hedges = scipy.zeros(nruns)
    pnls   = scipy.zeros(nruns)
    
    for step in range(nsteps):
        dzs = scipy.random.normal(0, sqrtdt, nruns)
        qs  = scipy.random.uniform(0, 1, nruns)
        ps  = scipy.random.binomial(1, 0.5, nruns) * 2 - 1 # +1 or -1 - trade quantities if a trade happens
        
        # check if there are client trades for each path
        
        indics  = scipy.less(qs, trade_prob)
        posns  += indics*ps
        trades += scipy.ones(nruns) * indics
        pnls   += scipy.ones(nruns) * indics * sprd_client * spots / 2.
        
        # check if there are hedges to do for each path
        
        if hedge_style == 'Zero':
            indics  = scipy.logical_or(scipy.less_equal(posns, -delta_lim), scipy.greater_equal(posns, delta_lim))
            pnls   -= scipy.absolute(posns) * indics * sprd_dealer * spots / 2.
            posns  -= posns * indics
            hedges += scipy.ones(nruns) * indics
        elif hedge_style == 'Edge':
            # first deal with cases where pos>delta_lim
            
            indics  = scipy.greater(posns, delta_lim)
            pnls   -= (posns - delta_lim) * indics * sprd_dealer * spots / 2.
            posns   = posns * scipy.logical_not(indics) + scipy.ones(nruns) * indics * delta_lim
            hedges += scipy.ones(nruns) * indics
            
            # then the cases where pos<-delta_lim
            
            indics  = scipy.less(posns, -delta_lim)
            pnls   -= (-delta_lim - posns) * indics * sprd_dealer * spots / 2.
            posns   = posns * scipy.logical_not(indics) + scipy.ones(nruns) * indics * (-delta_lim)
            hedges += scipy.ones(nruns) * indics
        else:
            raise ValueError('hedge_style must be "Edge" or "Zero"')
        
        # advance the spots and calculate period PNL
        
        dspots = vol * spots * dzs
        pnls  += posns * dspots
        spots += dspots
        
    return {'PNL': (pnls.mean(), pnls.std()), 'Trades': (trades.mean(), trades.std()), 'Hedges': (hedges.mean(), hedges.std())}


def run_simulation_slow(vol, lam, sprd_client, sprd_dealer, delta_lim, hedge_style, dt, nsteps, nruns, seed):
    '''Runs a Monte Carlo simulation and returns statics on PNL, client trades, and hedge trades.
    "_slow" because it does not use vectorized operations.
    
    vol:         lognormal volatility of the spot process
    lam:         Poisson process frequency
    sprd_client: fractional bid/ask spread for client trades. eg 1e-4 means 1bp.
    sprd_dealer: fractional bid/ask spread for inter-dealer hedge trades. eg 1e-4 means 1bp.
    delta_lim:   the delta limit at or beyond which the machine will hedge in the inter-dealer market
    hedge_style: 'Zero' or 'Edge', defining the hedging style. 'Zero' means hedge to zero position,
                 'Edge' means hedge to the nearer delta limit.
    dt:          length of a time step
    nsteps:      number of time steps for each run of the simulation
    nruns:       number of Monte Carlo runs
    seed:        RNG seed
    '''
    
    scipy.random.seed(seed)

    pnls   = []
    trades = []
    hedges = []

    trade_prob = 1 - exp(-lam * dt)
    sqrtdt     = sqrt(dt)
    
    for run in range(nruns):
        x  = 1
        rs = scipy.random.normal(0, sqrtdt, nsteps)
        qs = scipy.random.uniform(0, 1, nsteps)
        ps = scipy.random.binomial(1, 0.5, nsteps)

        pos = 0
        pnl = 0
        
        n_trades = 0
        n_hedges = 0

        for step in range(nsteps):
            # check if there's a client trade in the window. If so, assume it happens at the start of the window

            if qs[step] < trade_prob:
                if ps[step] == 0:
                    pos -= 1
                else:
                    pos += 1
                pnl += sprd_client * x / 2. # get paid the bid/ask spread on the unit notional of the trade
                
                n_trades += 1

            # check whether we need to hedge; if so, hedge back to zero position
            
            if hedge_style == 'Zero':
                if abs(pos) >= delta_lim:
                    pnl -= abs(pos) * sprd_dealer * x /2. # pay the bid/ask spread
                    pos  = 0
                    n_hedges += 1
            elif hedge_style == 'Edge':
                if abs(pos) > delta_lim:
                    if pos > delta_lim:
                        pnl -= (pos - delta_lim) * sprd_dealer * x / 2.
                        pos  = delta_lim
                    else: # pos<-delta_lim
                        pnl -= (-delta_lim - pos) * sprd_dealer * x/ 2.
                        pos  = -delta_lim

                    n_hedges += 1
            else:
                raise ValueError('hedge_style should be Zero or Edge')
            
            # step forward the spot

            dS = vol * x * rs[step]

            pnl += pos * dS
            x += dS

        pnls.append(pnl)
        trades.append(n_trades)
        hedges.append(n_hedges)
    
    pnls   = scipy.array(pnls)
    trades = scipy.array(trades)
    hedges = scipy.array(hedges)
    
    return {'PNL': (pnls.mean(), pnls.std()), 'Trades': (trades.mean(), trades.std()), 'Hedges': (hedges.mean(), hedges.std())}


def test_simulation():
    '''Little test function that demonstrates how to run this'''
    
    vol   = 0.1*sqrt(1/260.) # 10% annualized vol, converted to per-day vol using 260 days/year
    lam   = 1*60*60*24       # Poisson frequency for arrival of client trades: 1 per second, converted into per-day frequency to be consistent with vol

    sprd_client = 1e-4       # fractional full bid/ask spread for client trades
    sprd_dealer = 2e-4       # fractional full bid/ask spread for inter-dealer hedge trades

    hedge_style = 'Zero'     # 'Zero' means "hedge to zero position", or 'Edge' means "hedge to delta limit"

    delta_lim = 3.           # algorithm hedges when net delta reaches this limit

    dt       = 0.1/lam       # time step in simulation. Only zero or one client trades can arrive in a given interval.
    nsteps   = 500           # number of time steps in the simulation
    nruns    = 10000       # number of Monte Carlo runs
    seed     = 1             # Monte Carlo seed
    
    res = run_simulation_fast(vol,lam,sprd_client,sprd_dealer,delta_lim,hedge_style,dt,nsteps,nruns,seed)
    print('PNL Sharpe ratio             =',res['PNL'][0]/res['PNL'][1])
    print('PNL mean                     =',res['PNL'][0])
    print('PNL std dev                  =',res['PNL'][1])
    print('Mean number of client trades =',res['Trades'][0])
    print('SD number of client trades   =',res['Trades'][1])
    print('Mean number of hedge trades  =',res['Hedges'][0])
    print('SD number of hedge trades    =',res['Hedges'][1])

    
if __name__=="__main__":
    import time
    t1 = time.time()
    test_simulation()
    t2 = time.time()
    print('Elapsed time =',t2-t1,'seconds')
```