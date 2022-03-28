---
template: BlogPost
path: /impliedcorrelations
mockup: 
thumbnail:
date: 2019-08-20
name: FX Series - Implied Correlation in Cross Currency Markets
title: In this post of FX Series we will study implied correlations and compare them with pair volatility of a given currency
category: Quantitative Finance
description: 'Foreign Exchange implied correlation in the FX Spot Market.'
tags:
  - FX 
  - Trading
  - Spot
  - Quantitative Finance
---

In this post we will look at implied correlations and see how much moves in implied correlation contribute to moves in cross volatility, versus moves in the underlying USD-pair volatilities.

Consider the AUDJPY market, where the underlying USD pairs are AUDUSD and USDJPY.

For a given expiration tenor, one can calculate the market-implied correlation between moves in AUDUSD spot and USDJPY spot through the implied volatilities for the three pairs.

First step: we write the code to calculate these correlations in a window from 1Jan2007 to 31May2013. I have posted a spreadsheet with the ATM implied volatilities for AUDUSD, USDJPY, and AUDJPY for various expiration tenors on the class forum.

The function takes in the names of the three pairs (as strings like ‘AUDJPY’, ‘AUDUSD’, and ‘USDJPY’), a string tenor (like ‘3m’), a flag to define whether the cross spot is the product or the ratio of the two USD spots (which affects the sign of the correlation), and the start and end dates of the historical window.

It starts by loading the data for the ATM implied volatility for the three tenors from the spreadsheet into pandas DataFrames and then calculate a pandas DataFrame of implied correlations.

The next step: we use the correlation from date i, along with the implied volatilities for the USD pairs on date i+1, to predict the cross volatility on date i+1. We do this with the pandas DataFrames we have already created.

Finally,  we construct two DataFrames: one holding day-to-day changes in the cross ATM volatility, and one holding differences between the predicted cross volatility (assuming the implied correlation from the day before) and the true cross volatility.

The function prints out statistics on both those series.

Finally we run this for the following list of tenors: 1w, 1m, 6m, and 1y. 



```python
import pandas as pd
import numpy as np

data = pd.read_excel("fx_vol_data.xlsx", index="Date")
data.head()
```

|      |       Date | AUDUSD 1w | AUDUSD 1m | AUDUSD 6m | AUDUSD 1y | USDJPY 1w | USDJPY 1m | USDJPY 6m | USDJPY 1y | AUDJPY 1w | AUDJPY 1m | AUDJPY 6m | AUDJPY 1y |
| ---: | ---------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: | --------: |
|    0 | 2007-01-02 |    7.5000 |    7.1500 |    7.1750 |    7.3250 |    7.0000 |    6.5500 |   6.75000 |   6.90000 |      6.75 |       6.4 |      7.15 |      7.55 |
|    1 | 2007-01-03 |    7.9250 |    7.2750 |    7.3250 |    7.4000 |    6.8000 |    6.3000 |   6.62500 |   6.85000 |      6.75 |       6.4 |      7.15 |      7.55 |
|    2 | 2007-01-04 |    7.7500 |    7.2500 |    7.4100 |    7.4300 |    7.0018 |    6.7006 |   6.85045 |   6.92543 |      6.75 |       6.4 |      7.15 |      7.55 |
|    3 | 2007-01-05 |    7.7522 |    7.3007 |    7.4006 |    7.4506 |    6.8000 |    6.9500 |   6.90000 |   6.92000 |      7.50 |       7.5 |      7.30 |      7.40 |
|    4 | 2007-01-08 |    7.0500 |    7.3000 |    7.3000 |    7.4000 |    6.7500 |    6.7250 |   6.85000 |   6.95000 |      7.50 |       7.5 |      7.30 |      7.40 |

```python
def analyse(pairx, pair1, pair2, tenor):
    colx = pairx + ' ' + tenor
    col1 = pair1 + ' ' + tenor
    col2 = pair2 + ' ' + tenor

    # implied correlation 
    impCorr = (data[col1]**2 + data[col2]**2 - data[colx]**2) / (2.0 * data[col1] * data[col2])

    # predict corr for next day
    preVol = np.sqrt(data[col1]**2 + data[col2]**2 - 2 * impCorr.shift(1) * data[col1] * data[col2])
    
    #  differences between the predicted cross volatility and true
    diff = preVol - data[colx]
    
    # day-to-day changes
    dailyChange = data[colx].shift(1) - data[colx]

    return (diff.std(), diff.max(), diff.min(), dailyChange.std(), dailyChange.max(), dailyChange.min())
```



```python
tenors = ['1w', '1m', '6m', '1y']

for tenor in tenors:
    print(tenor)
    std1,max1,min1,std2,max2,min2 = analyse('AUDJPY','AUDUSD','USDJPY',tenor)
    print("    hedged std: ",std1)
    print("  unhedged std: ",std2)
    print("    hedged max: ",max1)
    print("  unhedged max: ",max2)
    print("    hedged min: ",min1)
    print("  unhedged min: ",min2)
    print("\n")
```



```
#OUTPUT
1w
    hedged std:  1.1860375166916477
  unhedged std:  1.7954802329602413
    hedged max:  14.951414531317791
  unhedged max:  15.88000000000001
    hedged min:  -13.923782223730102
  unhedged min:  -18.805


1m
    hedged std:  0.7424009483363391
  unhedged std:  1.0965227442381238
    hedged max:  7.8661789705795115
  unhedged max:  8.627499999999998
    hedged min:  -7.311626492498494
  unhedged min:  -12.5475


6m
    hedged std:  0.38322620425349
  unhedged std:  0.5750494585473307
    hedged max:  5.884164426715095
  unhedged max:  3.7225
    hedged min:  -3.4158506536508764
  unhedged min:  -4.922499999999999


1y
    hedged std:  0.31596331415981366
  unhedged std:  0.43140724936073804
    hedged max:  4.6054202940061835
  unhedged max:  2.482499999999998
    hedged min:  -3.265281784524058
  unhedged min:  -3.717500000000001
```

Conclusions

1. hedge for short tenor is better than long tenor, as max, min and std are all smaller than long tenor's
2. for long tenor like 6m or 1y, std is reduced but max and min increase.
3. assuming day-to-day vol not change is ok, though not work very well, but reduce risk somehow. Reduce about 25% - 33% std.