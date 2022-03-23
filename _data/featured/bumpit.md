---
template: BlogPost
path: /alpha
mockup:
thumbnail:
github: 
date: 2020-07
name: The ONLY article on ALPHAS you will get in the entirety of google which gives an insight into industrial quantitativ alpha generation algorithms
title: This is an algorithmic assessment of the paper published by a leading quant hedge fund.
category: Alphas
description: '1st place app idea would leverage latest of Azure and ML to support a $6.4 billion infrastructure issue.'
tags:
  - Alphas
  - Algorithmc Trading
  - Backtesting
---
# 1. Introduction to Quantitative ALPHA

In this post we will dive into the inner workings of quantitative hedge fund strategies. Many newly quantitative finance graduates or even enthusiasts do not understand how the quant hedge funds or even simple quant strategies work. Typical university curriculum teaches students to run programs in either MATLAB, Jupyer Notebook, or R-Studio IDE, with some exceptions using Wolfram Mathematica (which is the best)



## 1.1. Quantitative investment

Quantitative investment is to use the methods of modern statistics and mathematics to find the excess amount from the massive historical data. A variety of high-probability strategies for income, and disciplinedly follow the quantitative model built by these strategies to guide investment, Strive to obtain stable, sustainable, and above-average excess returns. Quantitative investment belongs to the category of active investment, the essence is the quantitative practice of qualitative investment is based on the ineffectiveness or weak effectiveness of the market.



## 1.2 Stock ALPHA multi-factor model

The model uses market inefficiency for statistical arbitrage. In general, it is very difficult to predict the trend of a stock, but if it is a basket of stocks and statistically predicts that the expectation is positive, the portfolio can obtain a stable excess over the long term

Earnings, we call this excess return the alpha of the stock. Multi-factor model is to find certain and stock returns. The most relevant series of factors. And according to the corresponding factors to construct a stock portfolio. Specifically, each factor is a mathematical expression or mathematical model for predicting the trend of stocks. It consists of various financial data, operators and functions. Number composition. The stock portfolio constructed by effective factors can obtain excess returns stably in the long term.



## 1.3 ALPHA factor system framework

**Figure 1** shows the analysis process of an ALPHA factor under our system framework. First, we will build a base The basic ALPHA model, which will score all stocks in the entire market. For example, we define `ALPHA= 1/close`, where close represents the closing price of the stock the previous day, and the score of the stock is the reciprocal of the closing price of the previous day. This score constitutes a raw alpha value of the model for all stocks. Further we optimize this original ALPHA value through various operation operators to obtain the final position (daily position). The back-testing system performs back-test analysis on factors based on the daily position of each stock.



<figcaption>ALPHA factor system framework</figcaption>



## 1.4 The long-short backtesting system

We use the long-short method to back-test the factors. Specifically, we will go through the operation process The absolute value of ALPHA is considered to be the position of the stock, and the sign represents the direction of long and short stocks. **Figure 2** gives the relationship between the position and direction of the stocks in the stock portfolio and their corresponding returns. By calculating the daily return of each stock we get the daily return of the factor stock portfolio, this time-based return sequence will be used to calculate the model ALPHA factor The effectiveness is evaluated.



<figcaption>Figure 2: The relationship between stock position, direction and return</figcaption>



## 1.5 Evaluation of factor effectiveness

The following are some commonly used indicators for evaluating the effectiveness of factors:

1. IR (Information Ratio) IR is equal to the average return divided by the volatility.
2. `sharpe (Sharp) sharpe = 15.8 ∗ IR`.
3. The annualized return of the return strategy
4. Fitness is a function to evaluate the performance of the strategy, which is composed of the combination of the profit of the strategy factor, the sharpness and the turnove Used to evaluate the effectiveness of the factor. A good strategy should have the highest possible returns, sharpness and low turnover.



# 2 How Quantitative Platforms Work



## 2.1 Platform installation

Most of these platforms are generally c++ files under folders and contains a set of hirarchy of sub-folders. Thus, only only needs to complie and execute the command codes from a unix like terminal or a windows command prompyt (this is also the reason why Linux is the go to OS for quantitative development, as it is fairly easier to install libraries directly to `/usr`  and execute the command).

The platform of these sorts have a simple hierarchy where the core pricer will be c++ based, while the script to call and run the instrument pricing algorithms will be in python. This is because python can not only run the c++ core programs, but also export the produced data in a `.csv` format and display the data in a desired graphical format using many of the python libraries.



## 2.2 Platform Hierarchy

The details of each file under the platform are as follows 

**folder**:

1. Core Folder: The strategy code is placed in the folder, which contains the strategy example CPP, which can be carried out by modifying the corresponding CPP Strategy development
2. Libraries: files after strategy compilation, used to link the files generated by strategy operation
3. Tools: Python (preferred language) toolbox for strategy result processing



**file**:

1. `.exe`: The execution file of the strategy operation (if you are using windows). This could also be a `.sln` file if the work environment is a Visual Studio IDE.
2. `config` : A general configuration file, used to configure various parameters of the strategy operation.



## 2.3 config file configuration

The following should give a general idea of how the file configurations are aligned and in what manner so that program runs optimally (file structure, especially file hierarchy is critical).

1. Macros defines macros, the absolute path of the reference used by the strategy
2. Universe configuration strategy operation start (startDate) and end time (endDate)
3. Modules module class, used to configure the data and ALPHA files of the strategy operation.
4. Portfolio backtest class, used to formulate the framework of strategy backtest and the operation operator used.



## 2.4 Strategy operation and results display

1. Strategy compilation: ./make
2. Strategy run: ./run config.xml (strategy configuration xml file)
3. Results display: 
   1. Strategy performance display: python tools/alpha stat.py pnl/strategy id (strategy pnl text File path)
   2. Relevance: /data/alphaSystem/release/tools/cor pnl/strategy id (strategy pnl file path)



# 3 Available data and functions



## 3.1 Data

The data currently available on the platform includes volume and price data, fundamental data, intraday data and other data. Every number according to the specific type, structure, name and description. This can be either a `.xls` with different column headers and multiple hierarchies (tabs within single sheet) within, or this can be a folder of `.csv` files arranged in a desired hierarchical manner.



## 3.2 Function







| Datetime |  Accel.X (m/s²) | Accel.Y (m/s²) | Accel.Z (m/s²) | Latitude | Longitude | Altitude (m) |
| --- | --- | --- | --- | --- | --- | --- |
2019-12-14 16:28:30:975 | -1.03 | -0.18 | 9.69 | 29.7955 | -95.568 | 6
2019-12-14 16:28:31:475 | -1.01 | -0.03 | 9.70 | 29.7955 | -95.568 | 6
2019-12-14 16:28:31:975 | -1.07 | -0.02 | 9.67 | 29.7955 | -95.568 | 6
2019-12-14 16:28:32:475 | -0.91 | -0.03 | 9.75 | 29.7955 | -95.568 | 6
2019-12-14 16:28:32:975 | -0.96 | -0.20 | 9.67 | 29.7955 | -95.568 | 6

![accelerometer data](assets/bumpit/fig1.svg)
<figcaption>Smartphone accelerometer data</figcaption>
