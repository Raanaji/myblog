---
template: BlogPost
path: /spline
mockup: 
thumbnail:
date: 2019-08-29
name: FX Series - Implementing a cubic spline interpolation for implied volatility
title: We implement a cubic spline interpolation for implied volatility vs strike which has non-standard boundary conditions
category: Quantitative Finance
description: 'Spline models for FX Modelling.'
tags:
  - FX 
  - Trading
  - Spline
  - Quantitative Finance
---

## Implementation

In this post we implement a cubic spline interpolation for implied volatility vs strike which has non-standard boundary conditions to give more intuitive volatility extrapolation.

Assuming We are given five implied volatilities, through , for five known strikes, through , for some known time to expiration , we set up the boundary condition of the cubic spline such that implied volatility reaches a constant value a certain distance beyond the edge points. The distance is defined by a parameter, the cubic spline extrapolation parameter, F. On the left side, for strikes less than the minimum marked strike , implied volatility reaches a constant value at strike : and on the right side, for strikes greater that the maximum marked strike , implied volatility reaches a constant value at strike :

We start with the [standard method for defining a cubic spline](https://gray.mgh.harvard.edu/attachments/article/337/Techniques%20of%20Proton%20Radiotherapy%20(09)%20Cubic%20Spline.pdf) but add in two extra points at and where we know that the slope of vol vs strike goes to zero, and the second derivative of vol vs strike goes to zero, but where we do not know the value of volatility (that comes out of the solution for the cubic spline). Of course the first and second derivatives of vol vs strike should be continuous across and .

Then we implement a Python class that is initialized with the five strikes, five implied volatilities, time to expiration, and cubic spline extrapolation parameter. As part of initialization it will need to solve a linear system to generate the cubic spline parameters, for which we can use the functions in the scipy or numpy packages. The class should include a “volatility” method that takes a strike and returns an interpolated volatility.

Finally, we generate plots of implied volatility vs strike for F=0.01 and F=10 for the following market: T=0.5y; ATM volatility of 8%, 25-delta risk reversal of 1%, 10-delta risk reversal of 1.8%, 25-delta butterfly of 0.25%, and 10-delta butterfly of 0.80%; spot=1; forward points=0; denominated discount rate=0%. We should return one chart with the two curves on it, showing implied volatility vs strike for all strikes between the 1-delta put and 1-delta call.



```python
import bisect
from scipy.stats import norm
import numpy as np
import matplotlib.pyplot as plt
```

```python
class CubicSpline:  
    def __init__(self, Ks, vols, T, F):
        
        K_min = Ks[0]  * np.exp(-F * vols[0]  * np.sqrt(T))
        K_max = Ks[-1] * np.exp( F * vols[-1] * np.sqrt(T))
        self.Ks = [K_min] + Ks + [K_max]
        
        M = np.array([[0.0 for i in range(24)] for j in range(24)])
        
        for i in range(5):
            M[4*i+2][4*i] = 1
            M[4*i+2][4*i+1] = self.Ks[i+1]
            M[4*i+2][4*i+2] = self.Ks[i+1]**2
            M[4*i+2][4*i+3] = self.Ks[i+1]**3
            
            M[4*i+3][4*i] = 1
            M[4*i+3][4*i+1] = self.Ks[i+1]
            M[4*i+3][4*i+2] = self.Ks[i+1]**2
            M[4*i+3][4*i+3] = self.Ks[i+1]**3
            M[4*i+3][4*i+4] = -1
            M[4*i+3][4*i+5] = -self.Ks[i+1]
            M[4*i+3][4*i+6] = -self.Ks[i+1]**2
            M[4*i+3][4*i+7] = -self.Ks[i+1]**3
            
            M[4*i+4][4*i+1] = 1
            M[4*i+4][4*i+2] = 2*self.Ks[i+1]
            M[4*i+4][4*i+3] = 3*self.Ks[i+1]**2
            M[4*i+4][4*i+5] = -1
            M[4*i+4][4*i+6] = -2*self.Ks[i+1]
            M[4*i+4][4*i+7] = -3*self.Ks[i+1]**2
            
            M[4*i+5][4*i+2] = 2
            M[4*i+5][4*i+3] = 6*self.Ks[i+1]
            M[4*i+5][4*i+6] = -2
            M[4*i+5][4*i+7] = -6*self.Ks[i+1]
        
        
        # left boundary
        M[0][1] = 1
        M[0][2] = 2*K_min
        M[0][3] = 3*K_min*K_min
        M[1][2] = 2
        M[1][3] = 6*K_min
        
        # right boundary
        M[24-2][24-3] = 1
        M[24-2][24-2] = 2*K_max
        M[24-2][24-1] = 3*K_max*K_max
        M[24-1][24-2] = 2
        M[24-1][24-1] = 6*K_max
        
        # y, where Mx=y  
        y = np.zeros(24)
        for i in range(5): # i=0,...,4
            y[4*i+2] = vols[i]
        
        # solve linear system
        x = np.linalg.solve(M,y)
        
        # different orders' coefficients
        self.x1 = [ x[4*i] for i in range(6) ]
        self.x2 = [ x[4*i+1] for i in range(6) ]
        self.x3 = [ x[4*i+2] for i in range(6) ]
        self.x4 = [ x[4*i+3] for i in range(6) ]
        
        
    def impvol(self, K):
        
        # if out of boundary
        if K < self.Ks[0]:
            K = self.Ks[0]
        elif K > self.Ks[-1]:
            K = self.Ks[-1]
            
        index = bisect.bisect_left(self.Ks, K)-1 
        if index < 0:
            index = 0
        elif index > 5:
            index = 5
            
        x1 = self.x1[index]
        x2 = self.x2[index]
        x3 = self.x3[index]
        x4 = self.x4[index]
        
        return x1 + x2*K + x3*K**2 + x4*K**3
```

```python
F1 = 0.01
F2 = 10
T = 0.5
spot = 1

atm = 0.08
reversal25 = 0.01
reversal10 = 0.018
butterfly25 = 0.0025
butterfly10 = 0.0080

# solve vol 
vol25c = atm + reversal25 / 2 + butterfly25
vol25p = atm - reversal25 / 2 + butterfly25
vol10c = atm + reversal10 / 2 + butterfly10
vol10p = atm - reversal10 / 2 + butterfly10

# solve strike
atmK = spot * np.exp(atm**2 * T /2.)
K25c  = spot * np.exp(vol25c**2  * T / 2. - vol25c * np.sqrt(T) * norm.ppf(0.25))
K25p  = spot * np.exp(vol25p**2  * T / 2. + vol25p * np.sqrt(T) * norm.ppf(0.25))
K10c  = spot * np.exp(vol10c**2  * T / 2. - vol10c * np.sqrt(T) * norm.ppf(0.10))
K10p  = spot * np.exp(vol10p**2  * T / 2. + vol10p * np.sqrt(T) * norm.ppf(0.10))

Ks = [K10p, K25p, atmK, K25c, K10c]
vols = [vol10p, vol25p, atm, vol25c, vol10c]

# create cubic spline classes
CS1 = CubicSpline(Ks, vols, T, F1)
CS2 = CubicSpline(Ks, vols, T, F2)
```

```python
# set grid to plot
c = spot * np.exp(vol10p ** 2 * T / 2. + vol10p * np.sqrt(T) * norm.ppf(0.01))
K_max = spot * np.exp(vol10c ** 2 * T / 2. - vol10c * np.sqrt(T) * norm.ppf(0.01))

# plot 1000 points for strike
n = 1000
dK = (K_max - K_min ) / (n - 1)
K_list,vol1_list,vol2_list = [], [], []

for i in range(n):
    K = K_min + i * dK
    K_list.append(K_min + i * dK)
    vol1_list.append(CS1.impvol(K) * 100)
    vol2_list.append(CS2.impvol(K) * 100)
```

```python
line1, = plt.plot(K_list, vol1_list, label='F=0.01')
line2, = plt.plot(K_list, vol2_list, label='F=10.0')
plt.xlabel('Strikes')
plt.ylabel('Implied Vol')
plt.legend()
plt.show()
```

![spline](/assets/spline/spline.png)

F represents the preset vol outside of the boundary. We can see for 𝐹 = 0.01, it flats very quickly. For  𝐹 = 10, it goes linearly because 10 is far away from the market point.