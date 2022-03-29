---
template: BlogPost
path: /spot
mockup: 
thumbnail:
date: 2019-08-24
name: FX Series - The Stochastic Spot/Vol Correlation Model
title: In this post of FX Series we will implement pricing for a jump diffusion model 
category: Quantitative Finance
description: 'Jump Diffusion Models in Foreign Exchange Markets.'
tags:
  - FX 
  - Trading
  - Jump Diffusion
  - Quantitative Finance
---

## Background

Consider a Merton jump diffusion model **𝑑𝑥(𝑡)=𝜇𝑑𝑡+𝜎𝑑𝑧+𝐽𝑑𝑞**, where **𝑥(𝑡)** is the log of spot, **𝜇** is the log-drift (a constant), **𝜎** is the diffusive volatility of spot (a constant), and **𝑞** is a Poisson process with frequency **𝜆**. That is, in the infinitesimal time period **𝑑𝑡**, **𝑑𝑞** is equal to 1 with probability **𝜆𝑑𝑡** and equal to 00 otherwise. **𝐽** is the jump that happens if the Poisson process fires; it is normally distributed with mean **𝐽𝑎** and variance **𝐽𝑣**. Jumps are independent of each other, independent of the Poisson process, and independent of the Brownian motion **𝑧**.

We’ll look at finding the characteristic function of **𝑥(𝑇)** for some future time **𝑇**, because with that we can calculate vanilla option prices with a single integration. Implement pricing for a jump diffusion model as in a  Merton jump diffusion model and/or the Heston Model.

One should have two functions: one that prices a vanilla option using the characteristic function we derive generally using Ito Lemma; and one that prices using conditional expectations, as defined below.

The first thing we need to do for either approach is figure out what drift **𝜇** to use to hit the forward to the expiration time 𝑇. For this, we need to calculate the value of a forward in the model; that is, the expected value of spot at time 𝑇.

For this we will use conditional expectations. That is, conditioned on exactly 𝑁 jumps happening by time 𝑇, 𝑥 is the sum of the normally-distributed Brownian motion plus the sum of the 𝑁 normally-distributed jumps; so, conditional on 𝑁 jumps, 𝑥 is normally distributed, and we can calculate the conditional expected value of spot.

For the next step, what we want is the unconditional expected value of 𝑆(𝑇), which we can identify with the forward (and then use to solve for the value of 𝜇 that makes the model match the forward). For this we can average over the number of jumps, weighting each by the probability of realizing that number of jumps. The probability of realizing 𝑁 jumps by time 𝑇 is of the form of a standard Poisson distribution.

The second function prices the vanilla option using the same conditional expectation approach we just used to calculate the forward. That is, conditioned on 𝑁 jumps, the distribution of spot is lognormal, and we can calculate the conditional expected value and the conditional variance of log(spot). That means, conditioned on 𝑁 jumps we can calculate the conditional option price using the Black-Scholes formula. Then we can average over the number of jumps, as above, and get the unconditional option price.

We should implement the two functions and make sure that they give the same price for the same option. (Not forgetting to discount future cashflows!) For the numerical integration, we use the `scipy` numerical integration package (eg `scipy.integrate.quad`).

Then, using either function, we generate a plot of implied volatility for the following market: spot = 1, time to expiration = 0.5y, forward points = 0.03, risk neutral discount rate 5%, 𝜎=7%, 𝐽𝑎=−4%, 𝐽𝑣=(15%)^2, 𝜆=3/year.

Discuss the impact the four model parameters (𝜎, 𝐽𝑎, 𝐽𝑣, and 𝜆) have on ATM vol, risk reversal, and butterfly.

```python
import numpy as np
from scipy.integrate import quad
from scipy.stats import norm
import matplotlib.pyplot as plt
```

```python
# return mu according to forward price
def mu_fun(S0, Ft, T, sigma, lamb, Ja, Jv):   
    return 1 / T * np.log(Ft / S0) - sigma**2/2 - lamb * (np.exp(Ja + Jv/2) - 1)

# normal cdf 
def norm_cdf(x):
    k = 1.0/(1.0+0.2316419*x)
    k_sum = k * (0.319381530 + k * (-0.356563782 + \
        k * (1.781477937 + k * (-1.821255978 + 1.330274429 * k))))
    if x >= 0.0:
        return (1.0 - (1.0 / ((2 * np.pi)**0.5)) * np.exp(-0.5 * x * x) * k_sum)
    else:
        return 1.0 - norm_cdf(-x)

# d in BS formula, when j=1, d-, when j=2, d+
def d_j(j, S, K, r, v, T):
    return (np.log(S/K) + (r + ((-1)**(j-1))*0.5*v*v)*T)/(v*(T**0.5))


def vanilla_call_price(S, K, r, v, T):
    return S * norm_cdf(d_j(1, S, K, r, v, T)) - \
        K*np.exp(-r*T) * norm_cdf(d_j(2, S, K, r, v, T))

def vanilla_put_price(S, K, r, v, T):
    return -S * norm_cdf(-d_j(1, S, K, r, v, T)) + \
        K*np.exp(-r*T) * norm_cdf(-d_j(2, S, K, r, v, T))
```

```python
def option_pricing_1(S0, K, T, sigma, r, rf, lamb, Ja, Jv, is_call):
    Ft = S0 * np.exp((r-rf)*T)
    mu = mu_fun(S0, Ft, T, sigma, lamb, Ja, Jv)
    
    def char_fun(theta):
        A = 1j * theta * mu * T - theta * theta * sigma * sigma * T / 2 \
          + lamb * T * (np.exp(1j * theta * Ja - theta * theta * Jv / 2) - 1)
        return np.exp(1j * theta * np.log(S0) + A)
    
    def integrand(theta):
        f = char_fun(theta)
        res = f * np.exp(-1j * theta * np.log(K/S0)) / (theta * theta + 1j * theta)
        return res.real
    
    ans, err = quad(integrand, 0, 1e3)

    price = Ft - K /2 - K / np.pi * ans
    
    # put-call parity
    if is_call == False:
        price -= Ft - K

    return price * np.exp(-r * T)
```

```python
def option_pricing_2(S0, K, T, sigma, r, rf, lamb, Ja, Jv, is_call):
    Ft = S0 * np.exp((r-rf)*T)
    mu = mu_fun(S0, Ft, T, sigma, lamb, Ja, Jv)
    
    def mean(N):
        return S0*np.exp(mu*T+0.5*sigma*sigma*T+(Ja+0.5*Jv)*N)
    
    def std(N):
        return np.sqrt((sigma*sigma*T+N*Jv)/T)
    
    def P(N):
        def factorial(n):
            if n == 0:
                return 1
            else:
                return n * factorial(n-1)
        return (lamb*T)**N/factorial(N)*np.exp(-lamb*T)
    
    price = 0
    
    for N in range(0,100):
        if is_call:
            price += vanilla_call_price(mean(N), K, 0, std(N), T)*P(N)
        else:
            price += vanilla_put_price(mean(N), K, 0, std(N), T)*P(N)
            
    return price * np.exp(-r * T)  
```

```python
S0 = 1
Ft = S0 + 0.03
T  = 0.5
sigma = 0.07
r = 0.05
lamb = 3
Ja = -0.04
Jv = 0.15**2
rf  = r - 1 / T * np.log(Ft / S0)
```

```python
# Test whether these two functions give same results
print("test Call princing")
for K in range(1,21):
    print("K = ", K/10, "difference = ", option_pricing_1(S0, K/10, T, sigma, r, rf, lamb, Ja, Jv, True)
          - option_pricing_2(S0, K/10, T, sigma, r, rf, lamb, Ja, Jv, True))
    
print("test Put princing")
for K in range(1,21):
    print("K = ", K/10, "difference = ", option_pricing_1(S0, K/10, T, sigma, r, rf, lamb, Ja, Jv, False)
          - option_pricing_2(S0, K/10, T, sigma, r, rf, lamb, Ja, Jv, False))
```

```
#OUTPUT
test Call princing
K =  0.1 difference =  2.7833291227352674e-13
K =  0.2 difference =  2.390043718492052e-11
K =  0.3 difference =  7.900035070562694e-10
K =  0.4 difference =  5.11257858271108e-09
K =  0.5 difference =  1.0561629681937745e-08
K =  0.6 difference =  4.46103348705762e-09
K =  0.7 difference =  2.716009528391794e-09
K =  0.8 difference =  -1.0290252816513856e-09
K =  0.9 difference =  -7.411948554914005e-10
K =  1.0 difference =  -6.0849522642847376e-09
K =  1.1 difference =  -4.1304928373453453e-08
K =  1.2 difference =  2.5043166502342062e-08
K =  1.3 difference =  3.1223931190593746e-08
K =  1.4 difference =  -2.6382271320693484e-08
K =  1.5 difference =  -2.447693208795093e-08
K =  1.6 difference =  -1.2033731461389557e-09
K =  1.7 difference =  1.8451172624067008e-08
K =  1.8 difference =  2.9561403287541813e-08
K =  1.9 difference =  3.316552131646562e-08
K =  2.0 difference =  3.1742944068055686e-08
test Put princing
K =  0.1 difference =  2.78138439671113e-13
K =  0.2 difference =  2.3900262124104905e-11
K =  0.3 difference =  7.900032890977514e-10
K =  0.4 difference =  5.112578592245283e-09
K =  0.5 difference =  1.0561629941387325e-08
K =  0.6 difference =  4.461033356628098e-09
K =  0.7 difference =  2.716009518417134e-09
K =  0.8 difference =  -1.029025363183389e-09
K =  0.9 difference =  -7.411948138580371e-10
K =  1.0 difference =  -6.0849522712236315e-09
K =  1.1 difference =  -4.130492838039235e-08
K =  1.2 difference =  2.5043166457239252e-08
K =  1.3 difference =  3.1223931329371624e-08
K =  1.4 difference =  -2.6382271312019867e-08
K =  1.5 difference =  -2.447693203677659e-08
K =  1.6 difference =  -1.2033731788818613e-09
K =  1.7 difference =  1.8451172922873127e-08
K =  1.8 difference =  2.956140332521784e-08
K =  1.9 difference =  3.31655211072146e-08
K =  2.0 difference =  3.1742944384127725e-08
```

```python
# implied vol calculation
def find_vol(target_value, call_put, S, K, T, r):
    
    n = norm.pdf
    N = norm.cdf

    def bs_price(cp_flag,S,K,T,r,v,q=0.0):
        d1 = (np.log(S/K)+(r+v*v/2.)*T)/(v*np.sqrt(T))
        d2 = d1-v*np.sqrt(T)
        if cp_flag == 'c':
            price = S*np.exp(-q*T)*N(d1)-K*np.exp(-r*T)*N(d2)
        else:
            price = K*np.exp(-r*T)*N(-d2)-S*np.exp(-q*T)*N(-d1)
        return price

    def bs_vega(cp_flag,S,K,T,r,v,q=0.0):
        d1 = (np.log(S/K)+(r+v*v/2.)*T)/(v*np.sqrt(T))
        return S * np.sqrt(T)*n(d1)

    MAX_ITERATIONS = 100
    PRECISION = 1.0e-5

    sigma = 0.5
    for i in range(0, MAX_ITERATIONS):
        price = bs_price(call_put, S, K, T, r, sigma)
        vega = bs_vega(call_put, S, K, T, r, sigma)

        price = price
        diff = target_value - price  # our root

        if (abs(diff) < PRECISION):
            return sigma
        sigma = sigma + diff/vega # f(x) / f'(x)

    # value wasn't found, return best guess so far
    return sigma
```

```python
strikes = []
vols = []
for K in np.arange(0.05,2,0.05):
    price = option_pricing_1(S0, K, T, sigma, r, rf, lamb, Ja, Jv, True)
    vol = find_vol(price, 'c', S0, K, T, r)
    strikes.append(K)
    vols.append(vol)

plt.figure(figsize=(12, 6))
plt.plot(strikes,vols)
plt.xlabel("strike")
plt.ylabel("impvol")
plt.title("implied volatility")
```

```
#OUTPUT
Text(0.5, 1.0, 'implied volatility')
```

![spotvol](/assets/spotvol/spotvol.png)