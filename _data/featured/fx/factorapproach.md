---
template: BlogPost
path: /factorapproach
mockup: 
thumbnail:
date: 2019-07-21
name: Factor-Base Approach Better For Reducing Dimensionality?
title: In this post of FX Series we will implement a factor model with hedges in a close market to determine whether it is better than ad-hoc method
category: Quantitative Finance
description: 'We will try to determine whether using a factor-based approach to reducing dimensionality is better than an ad hoc method.'
tags:
  - FX 
  - Factor Analysis
  - PCA 
  - Quantitative Finance
---

## Differences between two major modeling techniques



| PCA                                                          | Factor Models                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Look for most important (non-parametric) shocks that tend to drive moves in the whole curve | Assume correlated Brownian motions drive different factors   |
| Use historical daily interest rate spread moves, particularly covariances between them, to figure out the factors | Use non-linear root finding to find the model parameters that best match the historical dynamics |
| Picks largest few eigenvalues and approximating by linear combinations of corresponding eigenvectors | The number of parameters is the number of dimension          |



## Simulation

We will now will try to determine whether using a factor-based approach to reducing dimensionality is better than an ad hoc method.

We start by assuming a toy market: spot = 1, asset currency interest rate curve = Q(T) = flat at 3%, and denominated currency interest rate curve = R(T) = flat at 0%. We assume two benchmark dates, T1 = 0.25y and T2 = 1y; we will use forwards to those settlement dates to hedge the forward rate risk (or equivalently, the risk to moves in the asset currency interest rate) of our portfolio.

In the toy market, we assume that we know the dynamics of the asset currency interest rate:

where =1%/sqrt(yr), =0.8%/sqrt(yr), =0.5/yr, =0.1/yr, and =-0.4.

The portfolio to hedge has one position: a unit asset-currency notional of a forward contract settling at time T. We’ll try this for values of T in [0.1,0.25,0.5,0.75,1,2] to see how performance changes for portfolios with risk to different tenors.

We will try three different hedging strategies: one where we choose the hedge notionals (of forwards settling at times T1 and T2) based on the triangle shock we discussed in class (though as there are only two benchmarks here, the T1 shock will be flat before T1 and the T2 shock will be flat after T2); one where the notionals are set to hedge the actual two shocks from the factors described above; and lastly, one where we don’t hedge at all.

The result should show that setting hedge notionals based on the true factor shocks should provide a better hedge performance than based on the ad hoc triangle shocks. We will analyze just how much better that performance is.

We run a Monte Carlo simulation where we do the following on each run, for each value of T, for each of the three hedging strategies described above:

1. Construct a portfolio long 1 unit of the forward settling at time T
2. Add in the hedges: two forwards, settling at times T1 and T2, with notionals set to hedge the portfolio (either against the two triangle shocks or against the two factor shocks). Don’t bother adding the hedges in the third hedge scenario where we leave the portfolio unhedged.
3. Simulate the portfolio forward a time dt=0.001y. That will result in the asset-currency rates moving according to the factor model described above, which shocks the benchmark rates for tenors T1 and T2, and for the portfolio’s risk tenor T. Determine the PNL realized. 

Then construct the PNL distributions for the three hedging approaches. The unhedged version is the benchmark: we then compare how much more effectively the PNL standard deviation is reduced by hedging according to the true factors vs hedging according to the ad hoc triangle shocks.

```python
import math 
import scipy 
```

```python
Ts = [0.1,0.25,0.5,0.75,1,2]
hedgeTypes = ['Triangle', 'Factor', 'No hedge']
N = int(1e5) 
dt = 1e-3 

S0 = 1 
Q= 0.03
R = 0.00 
T1 = 0.25 
T2 = 1 

sigma1 = 0.01 
sigma2 = 0.008 
beta1 = 0.5 
beta2= 0.1 
rho = -0.4 

def simulation(T, dt, N, hedgeType):
    if hedgeType == 'Triangle':
        if T>T2:
            N1 = 0 
            N2 = 1 
        elif T<T1:
            N1 = 1 
            N2 = 0 
        else:
            N1 = (T2-T)/(T2-T1)
            N2 = (T-T1)/(T2-T1)
    elif hedgeType == 'Factor':
        N1 = (math.exp(-beta2 * T2 - beta1 * T)-math.exp(-beta1 * T2 - beta2 * T))/(math.exp(-beta1 * T1 - beta2 * T2)-math.exp(-beta1 * T2 - beta2 * T1))
        N2 = (math.exp(-beta1 * T1 - beta2 * T)-math.exp(-beta2 * T1 - beta1 * T))/(math.exp(-beta1 * T1 - beta2 * T2)-math.exp(-beta1 * T2 - beta2 * T1))
    else:
        N1 = 0 
        N2 = 0 
    
    
    N1 *= (T / T1) * math.exp(-Q * (T - T1))
    N2 *= (T / T2) * math.exp(-Q * (T - T2))

    dw1 = scipy.random.normal(0, math.sqrt(dt), N)
    dw2 = scipy.random.normal(0, math.sqrt(dt), N)
    dz1 = dw1
    dz2 = rho * dw1 + math.sqrt(1 - rho ** 2) * dw2

    dQT = sigma1 * math.exp(-beta1 * T)  * dz1 + sigma2 * math.exp(-beta2 * T)  * dz2
    dQ1 = sigma1 * math.exp(-beta1 * T1) * dz1 + sigma2 * math.exp(-beta2 * T1) * dz2
    dQ2 = sigma1 * math.exp(-beta1 * T2) * dz1 + sigma2 * math.exp(-beta2 * T2) * dz2

    pnls = S0 * math.exp(-Q * T) * (scipy.exp(-dQT * T) - 1)
    pnls -= N1 * S0 * math.exp(-Q * T1) * (scipy.exp(-dQ1 * T1) - 1)  
    pnls -= N2 * S0 * math.exp(-Q * T2) * (scipy.exp(-dQ2 * T2) - 1)

    return (pnls.mean(), pnls.std())
```

```python
for T in Ts:
    print('T = ',T)
    mean0, std0 = simulation(T,dt,N,'No hedge')
    mean1, std1 = simulation(T,dt,N,'Triangle')
    mean2, std2 = simulation(T,dt,N,'Factor')
    print("  No hedge std: ",std0)
    print("  Triangle std: ",std1,", relative ratio compare to no hedge: ",std1/std0)
    print("  Factor std: ",std2,", relative ratio compare to no hedge: ",std2/std0)
```

```
#OUTPUT
T =  0.1
  No hedge std:  3.035622658513886e-05
  Triangle std:  2.048759897031631e-06 , relative ratio compare to no hedge:  0.06749059838796362
  Factor std:  3.7456197441432754e-10 , relative ratio compare to no hedge:  1.2338884523865605e-05
T =  0.25
  No hedge std:  7.184032790960915e-05
  Triangle std:  0.0 , relative ratio compare to no hedge:  0.0
  Factor std:  0.0 , relative ratio compare to no hedge:  0.0
T =  0.5
  No hedge std:  0.00013182225499102116
  Triangle std:  1.8044887768154263e-06 , relative ratio compare to no hedge:  0.01368880222037118
  Factor std:  1.350852431285015e-09 , relative ratio compare to no hedge:  1.024752938247814e-05
T =  0.75
  No hedge std:  0.00018158365137431638
  Triangle std:  2.563531013452465e-06 , relative ratio compare to no hedge:  0.014117631152641624
  Factor std:  1.7348745976975343e-09 , relative ratio compare to no hedge:  9.55413433184723e-06
T =  1
  No hedge std:  0.00022605985126868445
  Triangle std:  0.0 , relative ratio compare to no hedge:  0.0
  Factor std:  0.0 , relative ratio compare to no hedge:  0.0
T =  2
  No hedge std:  0.0003626225746969014
  Triangle std:  0.00013112480805583724 , relative ratio compare to no hedge:  0.3616013376040863
  Factor std:  3.345546040106255e-08 , relative ratio compare to no hedge:  9.225972880763517e-05
```

Factor model hedge is almost perfect for all T. when 𝑇1<𝑇 <𝑇2, the triangle hedge performs good, the relative ratio is less then 2%. But when 𝑇 > 𝑇2 or 𝑇<𝑇1, performance is much worse. When 𝑇 = 𝑇2 or 𝑇=𝑇*1*, both methods perfectly hedge.