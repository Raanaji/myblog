---
template: BlogPost
path: /sklearn
mockup: 
thumbnail:
date: 2019-11-30
name: Machine Learning Series - Implementing RidgeCV and LassoCV
title: In this post of Machine Learning Series we will implement RidgeCV and LassoCV on selected data sets 
category: Machine Learning
description: 'Using the sklearn library in python we will implement some machine learning algorithms to fit data sets.'
tags:
  - Machine Learning 
  - RidgeCV
  - LassoCV
  - Data Engineering
---

## Implementation

We will use sklearn’s implementation of RidgeCV and LassoCV to fit data set 1 and 2 from above on 10,000 rows. If we  run a simple LassoCV fit with the default parameters, it will try different values of α, which is their version of a regularization parameter. After it fits, we can see the alpha values it tried (x.alphas ) as well as the value of α that it chose (x.alpha ). We then plot the out-of-sample error by generating a new test data set, also of size 10,000, for various values of α. 

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.linear_model import RidgeCV
from sklearn.linear_model import LassoCV
from sklearn.linear_model import Ridge
from sklearn.linear_model import Lasso
from sklearn.metrics import mean_squared_error
import torch	
```

#### Data Generation

```python
def generate_test_data_set1(n, f, sigma, test_sample=False):
    np.random.seed(0)
    true_betas = np.random.randn(f)

    np.random.seed(1 if test_sample else 2)
    X = np.random.randn(n, f)
    Y = np.random.randn(n) * sigma + X.dot(true_betas)

    return (X, Y)


def generate_test_data_set2(n, f, sigma, test_sample=False):
    np.random.seed(0)
    true_betas = np.random.randn(f)
    true_betas[-f // 2:] = 0

    np.random.seed(1 if test_sample else 2)
    X = np.random.randn(n, f)
    Y = np.random.randn(n) * sigma + X.dot(true_betas)

    return (X, Y)
```

```python
#generate 4 data sets and corresponding 4 test sets

data_set11=generate_test_data_set1(10000,5,1)
data_set21=generate_test_data_set2(10000,5,1)
test_set11=generate_test_data_set1(10000,5,1,True)
test_set21=generate_test_data_set2(10000,5,1,True)

data_set12=generate_test_data_set1(10000,20,100)
data_set22=generate_test_data_set2(10000,20,100)
test_set12=generate_test_data_set1(10000,20,100,True)
test_set22=generate_test_data_set2(10000,20,100,True)
```

```python
alphas=[]
i=1e-6
while i<=1e8:
    alphas.append(i)
    i*=10
```

```python
def ridge_lasso_cv(data_set,alphas=alphas):
    ridgeCV = RidgeCV(alphas).fit(data_set[0],data_set[1])
    lassoCV = LassoCV(alphas=alphas,cv=5).fit(data_set[0],data_set[1])

    print("alphas: ",alphas)
    print("Ridge CV:","\n  chosen alpha: ",ridgeCV.alpha_)
    print("Lasso CV:","\n  chosen alpha: ",lassoCV.alpha_)
    
    return (lassoCV.alpha_,ridgeCV.alpha_)
```

In Ridge, we use leave one out cross validation

In Lasso, we use 5-fold corss validation

```python
def error_plot(data_set,test_set,lasso_cv_alpha,ridge_cv_alpha,title=None):
    mse_ridge_test=[]
    mse_lasso_test=[]
    mse_ridge_train=[]
    mse_lasso_train=[]
    lasso_in_err,lasso_out_err,ridge_in_err,ridge_out_err = 0,0,0,0
    
    for alpha in alphas:
        ridge=Ridge(alpha=alpha).fit(data_set[0],data_set[1])
        mse_ridge_train.append(mean_squared_error(data_set[1],ridge.predict(data_set[0])))
        mse_ridge_test.append(mean_squared_error(test_set[1],ridge.predict(test_set[0])))

        lasso=Lasso(alpha=alpha).fit(data_set[0],data_set[1])
        mse_lasso_train.append(mean_squared_error(data_set[1],lasso.predict(data_set[0])))
        mse_lasso_test.append(mean_squared_error(test_set[1],lasso.predict(test_set[0])))
        
        if alpha==lasso_cv_alpha:
            lasso_in_err,lasso_out_err = mse_lasso_train[-1], mse_lasso_test[-1]
        if alpha==ridge_cv_alpha:
            ridge_in_err,ridge_out_err = mse_ridge_train[-1], mse_ridge_test[-1]
            

    lg_alphas=[np.log10(alpha) for alpha in alphas]

    plt.figure(figsize=(14,7))
    plt.plot(lg_alphas,mse_ridge_train,"b+--",label="ridge in-sample")
    plt.plot(lg_alphas,mse_lasso_train,"r+--",label="lasso in-sample")
    plt.plot(lg_alphas,mse_ridge_test,'bo--',label="ridge out-of-sample")
    plt.plot(lg_alphas,mse_lasso_test,"ro--",label="lasso out-of-sample")
    plt.xlabel("lg(alpha)")
    plt.ylabel("mse")
    plt.title(title)
    plt.legend()
    plt.show()
    
    return (lasso_in_err,lasso_out_err,ridge_in_err,ridge_out_err)
```

#### Data Set I with parameter I

```python
alpha11_lasso,alpha11_ridge = ridge_lasso_cv(data_set11)
```

```
alphas:  [1e-06, 9.999999999999999e-06, 9.999999999999999e-05, 0.001, 0.01, 0.1, 1.0, 10.0, 100.0, 1000.0, 10000.0, 100000.0, 1000000.0, 10000000.0, 100000000.0]
Ridge CV: 
  chosen alpha:  0.1
Lasso CV: 
  chosen alpha:  1e-06
```

```python
err11_lasso_in,err11_lasso_out,err11_ridge_in,err11_ridge_out = error_plot(
    data_set11,test_set11,alpha11_lasso,alpha11_ridge, "data set I with parameter I mse against lg(alpha)")
```

![1](/assets/ml/1.png)

#### Data Set I with parameter II

```python
alpha12_lasso,alpha12_ridge = ridge_lasso_cv(data_set12)
```

```
alphas:  [1e-06, 9.999999999999999e-06, 9.999999999999999e-05, 0.001, 0.01, 0.1, 1.0, 10.0, 100.0, 1000.0, 10000.0, 100000.0, 1000000.0, 10000000.0, 100000000.0]
Ridge CV: 
  chosen alpha:  10000.0
Lasso CV: 
  chosen alpha:  1.0
```

```python
err12_lasso_in,err12_lasso_out,err12_ridge_in,err12_ridge_out = error_plot(
    data_set12,test_set12,alpha12_lasso,alpha12_ridge, "data set I with parameter II mse against lg(alpha)")
```

![2](/assets/ml/2.png)

#### Data Set II with parameter I

```python
alpha21_lasso,alpha21_ridge = ridge_lasso_cv(data_set21)
```

```
alphas:  [1e-06, 9.999999999999999e-06, 9.999999999999999e-05, 0.001, 0.01, 0.1, 1.0, 10.0, 100.0, 1000.0, 10000.0, 100000.0, 1000000.0, 10000000.0, 100000000.0]
Ridge CV: 
  chosen alpha:  1.0
Lasso CV: 
  chosen alpha:  0.001
```

```python
err21_lasso_in,err21_lasso_out,err21_ridge_in,err21_ridge_out = error_plot(
    data_set21,test_set21,alpha21_lasso,alpha21_ridge, "data set II with parameter I mse against lg(alpha)")
```

![3](/assets/ml/3.png)

#### Data Set II with parameter I

```python
alpha22_lasso,alpha22_ridge = ridge_lasso_cv(data_set22)
```

```
alphas:  [1e-06, 9.999999999999999e-06, 9.999999999999999e-05, 0.001, 0.01, 0.1, 1.0, 10.0, 100.0, 1000.0, 10000.0, 100000.0, 1000000.0, 10000000.0, 100000000.0]
Ridge CV: 
  chosen alpha:  100000000.0
Lasso CV: 
  chosen alpha:  100000000.0
```

```python
err22_lasso_in,err22_lasso_out,err22_ridge_in,err22_ridge_out = error_plot(
    data_set22,test_set22,alpha22_lasso,alpha22_ridge, "data set II with parameter II mse against lg(alpha)")
```

![4](/assets/ml/4.png)

```python
comparison={'redundant beta':['False','False','True','True'],
     'f':[5,20,5,20],
     'sigma':[1,100,1,100],
     'lasso cv alpha':[alpha11_lasso,alpha12_lasso,alpha21_lasso,alpha22_lasso],
     'lasso in-sample err':[err11_lasso_in,err12_lasso_in,err21_lasso_in,err22_lasso_in],
     'lasso out-of-ample err':[err11_lasso_out,err12_lasso_out,err21_lasso_out,err22_lasso_out],
     'ridge cv alpha':[alpha11_ridge,alpha12_ridge,alpha21_ridge,alpha22_ridge],
     'ridge in-sample err':[err11_ridge_in,err12_ridge_in,err21_ridge_in,err22_ridge_in],
     'ridge out-of-ample err':[err11_ridge_out,err12_ridge_out,err21_ridge_out,err22_ridge_out]}
comparison=pd.DataFrame(comparison)
```

```python
comparison
```

|      | redundant beta |    f | sigma | lasso cv alpha | lasso in-sample err | lasso out-of-ample err | ridge cv alpha | ridge in-sample err | ridge out-of-ample err |
| ---: | -------------: | ---: | ----: | -------------: | ------------------: | ---------------------: | -------------: | ------------------: | ---------------------: |
|    0 |          False |    5 |     1 |   1.000000e-06 |            0.997691 |               1.010031 |            0.1 |            0.997691 |               1.010030 |
|    1 |          False |   20 |   100 |   1.000000e+00 |         9946.885342 |           10167.840990 |        10000.0 |         9944.333680 |           10164.693756 |
|    2 |           True |    5 |     1 |   1.000000e-03 |            0.997695 |               1.010077 |            1.0 |            0.997691 |               1.010033 |
|    3 |           True |   20 |   100 |   1.000000e+08 |         9955.339280 |           10168.207859 |    100000000.0 |         9955.335464 |           10168.206320 |

Greater f means more features, which leads to higher alpha value. This makes sense because we need to penalize more to avoid over-fitting, since we have more data. Greater sigma leads to very large both in-sample and out-of-sample error, this is consistent with definition of mse.

Redundant betas leads to greater alpha, as we need to shrink more or drop the redundant features by greater penalty.

Lasso and ridge under best cross validation alpha performs equally well, we can see for same data set, their error are almost same.
