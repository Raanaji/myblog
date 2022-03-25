---
template: BlogPost
path: /alpha
mockup:
thumbnail:
github: 
date: 2020-07
name: The only article on quantitative alpha you will need!
title: An algorithmic assessment of the quant ALPHA.
category: Quantitative Finance
description: 'Quantitative discussion on ALPHAS'
tags:
  - Alphas 101
  - Algorithmc Trading
  - Backtesting
---
# 1 Introduction to the concept of ALPHA in quantitative finance

In this post we will dive into the **inner workings of quantitative hedge fund strategies**. Most of the material here are from my own experience of working as a freelance quant on understanding and designing strategies which work on core pricing system principles, unlike the front-end targeting markdown notebook format which is visually impressive, but achieves little except presenting data beautifully.

Many newly quantitative finance graduates or even enthusiasts do not understand how the quant hedge funds or even simple quant strategies work. Typical university curriculum teaches students to run programs in either MATLAB, Jupyer Notebook, or R-Studio IDE, with some exceptions using Wolfram Mathematica (which is the best). Through this discussion, we will delve deeper into understanding how traditional and boutique quant shops manage their day to day trading strategies and in what format.

The following structure will be familiar to people using Quantlib library. Most of what I have here is pseudo-code.



## 1.1 Quantitative investment

Quantitative investment is to use the methods of modern statistics and mathematics to find the excess amount from the massive historical data. A variety of high-probability strategies for income, and disciplinedly follow the quantitative model built by these strategies to guide investment, Strive to obtain stable, sustainable, and above-average excess returns. Quantitative investment belongs to the category of active investment, the essence is the quantitative practice of qualitative investment is based on the ineffectiveness or weak effectiveness of the market.



## 1.2 Stock ALPHA multi-factor model

The model uses market inefficiency for statistical arbitrage. In general, it is very difficult to predict the trend of a stock, but if it is a basket of stocks and statistically predicts that the expectation is positive, the portfolio can obtain a stable excess over the long term

Earnings, we call this excess return the alpha of the stock. Multi-factor model is to find certain and stock returns. The most relevant series of factors. And according to the corresponding factors to construct a stock portfolio. Specifically, each factor is a mathematical expression or mathematical model for predicting the trend of stocks. It consists of various financial data, operators and functions. Number composition. The stock portfolio constructed by effective factors can obtain excess returns stably in the long term.



## 1.3 ALPHA factor system framework

**Figure 1** shows the analysis process of an ALPHA factor under our system framework. First, we will build a base The basic ALPHA model, which will score all stocks in the entire market. For example, we define `ALPHA= 1/close`, where close represents the closing price of the stock the previous day, and the score of the stock is the reciprocal of the closing price of the previous day. This score constitutes a raw alpha value of the model for all stocks. Further we optimize this original ALPHA value through various operation operators to obtain the final position (daily position). The back-testing system performs back-test analysis on factors based on the daily position of each stock.

![alpha1](/assets/alpha/alpha1.png)

<figcaption>ALPHA factor system framework</figcaption>



## 1.4 The long-short backtesting system

We use the long-short method to back-test the factors. Specifically, we will go through the operation process The absolute value of ALPHA is considered to be the position of the stock, and the sign represents the direction of long and short stocks. below table gives the relationship between the position and direction of the stocks in the stock portfolio and their corresponding returns. By calculating the daily return of each stock we get the daily return of the factor stock portfolio, this time-based return sequence will be used to calculate the model ALPHA factor The effectiveness is evaluated.



| Position | Name  | PnL if the stock price raises by 1% | PnL if the stock price declines by 1% |
| -------- | ----- | ----------------------------------- | ------------------------------------- |
| $100                              | Long                              |$1|-$1|
| -$100                                | Short                               |-$1|$1|



## 1.5 Evaluation of factor effectiveness

The following are some commonly used indicators for evaluating the effectiveness of factors:

1. IR (Information Ratio) IR is equal to the average return divided by the volatility.
2. `sharpe (Sharp) sharpe = 15.8 ∗ IR`.
3. The annualized return of the return strategy
4. Fitness is a function to evaluate the performance of the strategy, which is composed of the combination of the profit of the strategy factor, the sharpness and the turnove Used to evaluate the effectiveness of the factor. A good strategy should have the highest possible returns, sharpness and low turnover.



# 2 How Quantitative Platforms Work



## 2.1 General Platform installation

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

Through the method call of `QL Oputils::`, for example, `QL Oputils::std(x)` is to find the variance of the sequence x, the following is Some commonly used functions currently available on the platform:

1. `const bool QL less (const T &x, const T &y)` 
   *Note*: compare x, y if x<y, then true, otherwise false

2. `int sign(TA)` 
   *Description*: Return the symbol of A

3. `float decay linear(vector<float> &x)` 
   *Description*: Returns the value of the sequence x subjected to linear decay processing.  `Decay linear(x)=(x[0] ∗ n + x[1] ∗ (n − 1) +... + x[n − 1])/(n + (n − 1) + ... + 1)`.

4.  `float decay exp(vector<float> &x)` 
   *Description*: Returns the value of the sequence x subjected to linear decay processing. `Decay exp(x, f)=(x[0] + x[1] ∗ f + ... + x[n −\1) ∗ f n-1 )/(1 + f + ... + f n-1 ).`

5. `int rank(vector<float> &x)` 
   *Description*: Perform rank processing on the sequence x and return the number of effective values of the sequence. For example, the closing price of 6 stocks is [20.2, 15.6, 10.0, 5.7, 50.2, 18.4], the sequence after rank is [0.8, 0.4, 0.2, 0.0, 1.0, 0.6].

6. `void power(vector<float> &x, float e, bool dorank = true)` 

   *Note*: To perform power processing on sequence x, dorank processing will be done first by default, that is, sequence rank first and then power. For example, the closing prices of 6 stocks are [20.2, 15.6, 10.0, 5.7, 50.2, 18.4], and the sequence after rank is [0.8, 0.4, 0.2,0.0, 1.0, 0.6]. Suppose you set e to 2, then power will be [0.64, 0.16, 0.04, 0, 1, 0.36]. If you don’t want To do rank processing, `set dorank = false`.

7. `float sum (vector<float> &x)`
   Explanation: Find the sum of the sequence x.

8. `float product (vector<float> &x)`
   Explanation: Find the product of the sequence x.

9. `float mean (vector<float> &x)`
   Description: Find the average value of the sequence x.

10. ` float median (vector<float> &x)`
    Explanation: Find the median of the sequence x.

11. `float std (vector<float> &x)`
    Explanation: Find the variance of the sequence x.

12. `float skew (vector<float> &x)`
    Explanation: Find the kurtosis of the sequence x.

13. `float kurtosis (vector<float> &x)`
    Explanation: Find the skewness of the sequence x.

14. `float corr(vector<float> &x, vector<float> &y)`
     Description: Calculate the correlation between the sequence x and y.

15. `llv (T first, T last)`
    Description: Find the minimum value of the sequence. Example: For example, if there is a vector x, then `llv(x.begin(),x.end())` will return the sequence x Minimum value.

16. `hhv (T first, T last)`
    Description: Find the maximum value of the sequence. Example: For example, if there is a vector x, then `hhv(x.begin(),x.end())` will return the sequence x Maximum value.

17. `float beta range (T x first, T x last, T y first, T y last)`
    Description: Calculate the beta of sequence x relative to sequence y. Example: beta range(x.begin(),x.end(),y.begin(),y.end()).

18. `float slop (T first, T last)`
    Description: Find the slope of the sequence.



## 3.3 Global functions



Through the method call of `GLOBALFUNC::`, such as `GLOBALFUNC::isnan(x)` is to judge whether x For nan, the following are some commonly used global functions currently available on the platform:

1. `bool isnan(T a)`
   Description: Determine whether a is nan
2. `bool isinf(T a)`
   Description: Determine whether a is inf
3. `bool iserr(T a)`
   Description: Determine whether a is err (whether it is nan or inf)
4. `bool inline equal(float x, float y)`
   Description: Determine whether x and y are equal
5. `int inline findInstrumentIdx(const string& sInstrument)` 
   Description: Find the index of a stock in the stock pool



## 3.4 Global variables

Called by the `GLOBAL::` method, for example, GLOBAL::Dates(di) is the date corresponding to `di:`

1. Dates
   Description: `Date array, GLOBAL::Dates(di)` is the date corresponding to di
2. Instruments
   Description: Date array, `GLOBAL::Instruments(ii)` is the corresponding stock code of ii



## 3.5 Date functions

Through the method call of QLCalendar::, for example, QLCalendar::getDi(date) is the date corresponding to the return date Di:

1. `int inline getDi(int date)`
    Description: get the di corresponding to date
2. `int inline distance day(int date a, int date b)`
   Explanation: Get the number of days between two dates (regardless of whether it is a trading day), and get the absolute value, and there will be no negative production Give birth
3. `int inline getDateAfterDays(int date, int days)`
   Description: How many days backward the date. If it is negative, it is forward
4. `int inline distance tradDay(int date a, int date b)`
    Description: The number of trading days between two dates, the absolute value is obtained
5. `int inline distance tradDay(int date a, int date b)`
    Note: The number of trading days after the trading day. If it is negative, it is forward
6. `int inline getDate(int di)` 
   Description: date obtained by di
7. `int inline returnWeekDay(int date)` 
   Explanation: get date is the day of the week





# 4 Operation operator description

The common operation operators of the platform are as follows

1. Decay decays the ALPHA value. The optional parameter days represents the number of days used for decay. `Decay(x, n) = (x[date] * n + x[date-1] * (n-1) + ... + x[date – n-1]) / (n + (n-1) + ... + 1)` Where n represents days

2. Power performs exponential processing on the ALPHA value, the optional parameter exp, represents the power of the exponent, and the exponent is used by default RANK processing will be done before processing

3. Truncate truncates the ALPHA value, the optional parameter maxPercent, represents the percentage of truncation Ratio, if the ALPHA value exceeds the maxPercent of the sum of all ALPHA values, it is equal to the sum of all ALPHA values `MaxPercent`.

4. `IndNeut` does industry-neutral processing on the ALPHA value, and the optional parameter group represents industry-neutral specific Category.

   

# 5 Preparation of ALPHA strategy

A typical strategy writing process is shown in Figure 4. First, a rough idea is generated through mathematical modeling The ALPHA of each stock is processed through the operation operator to obtain the position of each stock at the time of simulation, and the backtest system is used Calculate the daily profit of the strategy and generate various indicators of the strategy. Then optimize and improve the strategy according to various indicators.

![alpha3](/assets/alpha/alpha3.png)

<figcaption>Figure 3: Typical strategy writing process</figcaption>



## 5.1 Data declaration and access

1. Data declaration The data declaration is under private. For example, if you need the daily opening data of the stock, you can say `const QL MATRIX<QL FLOAT> &open`
2. Data acquisition Data acquisition is under public. For example, if you need daily opening data of stocks, you can obtain it as follows take. `open(dr.getData<QL MATRIX<QL FLOAT>>("adj open"))`



## 5.2 Strategy writing

The writing of the strategy is realized by the replication function void generate(int di), where di represents a specific day.

The specific logic is to score each stock in the stock pool (the score is the ALPHA value of the stock), that is, to create a stock The functional logical relationship between scores and data.

The precautions for the writing process are:

1. Give every stock a score as much as possible, if you can't give a score, give a score of 0
2. The current system adopts the delay 1 mode, and future data cannot be used to score stocks, that is, the ALPHA on the di day can only Use di-1 day or earlier data. The description of delay 1 mode is shown in **Figure 4**.



<img src="/assets/alpha/alpha4.png" style="zoom:50%;" />



<figcaption>Figure 4: Delay 1 process</figcaption>



The alpha 62 strategy code is shown in **Figure 5**:



```c++
#include "include/qlSim.h"
namespace
{
class alpha62: public AlphaBase
{
    public:
    explicit alpha62(XMLCONFIG::Element *cfg):
        AlphaBase(cfg), 
        vwap(dr.getData<QL_MATRIX<QL_FLOAT>>("adj_vwap")), 
        open(dr.getData<QL_MATRIX<QL_FLOAT>>("adj_open")),
        vol(dr.getData<QL_MATRIX<QL_FLOAT>>("volume")),
        high(dr.getData<QL_MATRIX<QL_FLOAT>>("adj_high")),
        low(dr.getData<QL_MATRIX<QL_FLOAT>>("adj_low")),
        Num1(cfg->getAttributeIntDefault("para1",20)),
	Num2(cfg->getAttributeIntDefault("para2",22)),
	Num3(cfg->getAttributeIntDefault("para3",9))
	
    {
        adv20.resize (GLOBAL::Dates.size());

        for (int di = 0; di< GLOBAL::Dates.size(); ++di)
        {
            adv20[di].resize(GLOBAL::Instruments.size(),nan);
        }
    }
    void generate(int di) override
    {
    	vector<float> rank1(GLOBAL::Instruments.size(),nan);
    	vector<float> rank2(GLOBAL::Instruments.size(),nan);
    	vector<float> open2_rank(GLOBAL::Instruments.size(),nan);
    	vector<float> med_rank(GLOBAL::Instruments.size(),nan);
    	vector<float> high_rank(GLOBAL::Instruments.size(),nan);

       for(int ii = 0; ii < GLOBAL::Instruments.size(); ++ ii)
        {
            if((valid[ di ][ ii ]))
            {

                vector<float> c1(Num1,nan);
                for (int j = 0; j < Num1; ++ j)
                {
                    c1[j] = vol[di - delay - j][ii];
                }
                adv20[di-delay][ii] = QL_Oputils::mean(c1);
            }
        }

		for(int ii = 0; ii < GLOBAL::Instruments.size(); ++ ii)
        {
            if((valid[ di ][ ii ]))
            {	
            	vector<float> c3(Num3,0);
            	vector<float> c4(Num3,nan);


				for (int j =0; j<Num3; ++j)
	    		  {
	    		  	c3[j] = 0.0;
					for (int k = 0; k< Num2; ++k)
					{
		  				c3[j] += adv20[di-delay-k-j][ii];
					}

					c4[j] = vwap[di-delay-j][ii];
	      		  }

	      		rank1[ii] = QL_Oputils::corr(c3,c4);
	 	 	}
		}

		QL_Oputils::rank(rank1);
		
		 for(int ii = 0; ii < GLOBAL::Instruments.size(); ++ ii)
        {
            if((valid[ di ][ ii ]))
            {
            	open2_rank[ii] = 2.0 * open[di-delay][ii];
            	med_rank[ii] = 0.5 * (high[di-delay][ii]+low[di-delay][ii]);
            	high_rank[ii] = high[di-delay][ii];
            }
    	}

		QL_Oputils::rank(open2_rank);		
		QL_Oputils::rank(med_rank);
		QL_Oputils::rank(high_rank);

		 for(int ii = 0; ii < GLOBAL::Instruments.size(); ++ ii)
        {
            if((valid[ di ][ ii ]))
            {
            	if (open2_rank[ii]<(med_rank[ii]+high_rank[ii]))
            		{rank2[ii]=1;}
            	else
            		{rank2[ii]=0;}
            }
    	}

    	QL_Oputils::rank(rank2);


        for(int ii = 0; ii < GLOBAL::Instruments.size(); ++ ii)
        {
            if((valid[ di ][ ii ]))
            {
    
	    
	    		if (rank1[ii]<rank2[ii])
	    			{alpha[ii]=-1;}
	    		else
	    			{alpha[ii]= 0;}
	    }
        }
        return;
    }
    
    
    void checkPointSave(boost::archive::binary_oarchive &ar) 
    {
        ar & *this;
    }
    void checkPointLoad(boost::archive::binary_iarchive &ar)
    {
        ar & *this;
    }
    std::string version() const
    {
        return GLOBALCONST::VERSION;
    }
    private:
        friend class boost::serialization::access;
        template<typename Archive>
        void serialize(Archive & ar, const unsigned int/*file_version*/)
        {
        }
	const QL_MATRIX<QL_FLOAT> &vwap;
	const QL_MATRIX<QL_FLOAT> &open;
	const QL_MATRIX<QL_FLOAT> &vol;
	const QL_MATRIX<QL_FLOAT> &high;
	const QL_MATRIX<QL_FLOAT> &low;
    vector<vector<float> > adv20;
        int Num1; 
	int Num2;
	int Num3;
};
}
extern "C"
{
    AlphaBase * createStrategy(XMLCONFIG::Element *cfg)
    {
        AlphaBase * str = new alpha62(cfg);
        return str;
    }
}


```



<figcaption>Figure 5: Alpha 62 strategy code</figcaption>



## 5.3 Naming convention

1. Cpp file naming format: username alphaname.cpp
2. xml file naming format: config.username alphaname.xml
3. The naming format of xml internal alpha id is username alphaname 
4. Teamwork strategy naming usernameA usernameB alphaname





# 6 Evaluation of strategies

The performance of the strategy is realized by calling the alpha stat.py file in the python toolbox. In the above example, 5day- The return result is shown in **Figure 6**:

Among them, dates represents the backtest period. The current backtest period in the strategy sample is from 200601 to 201501. long, short generation Shows long and short positions, ret represents revenue, turnover represents turnover rate, ir represents information ratio, and drawdown represents Maximum drawdown, win represents the winning rate, long num, short num represent the average number of long stocks and average short stocks

Number of votes. fitness stands for strategy score. Up days and down days represent the longest continuous rise and the longest continuous decline day.



![](assets/alpha/alpha6.png)



<figcaption>Figure 6: 5 day-return strategy performance</figcaption>



The relevance of the strategy is evaluated through the command /data/alphaSystem/release/tools/cor pnl/strategy id (strategy id) The pnl file path is omitted), and the 5day-return correlation evaluation result is shown in **Figure 7**. Cor, ISSharp, OS-Sharp and Fitness respectively gave the correlations of the five strategies with the highest and the lowest correlations with strategies.

General, Sharpe and Fitness scores outside the strategy sample. Count and count better are given in the strategy and strategy library respectively The number of strategies at each level of relevance, and the number of strategies that are better than the strategy.



<img src="/assets/alpha/alpha7.png" style="zoom: 33%;" />



<figcaption>Figure 7: Correlation performance of 5 day-return</figcaption>



# 7 Optimization of ALPHA strategy



## 7.1 Optimization method

1. Idea optimization, that is, the optimization of the model algorithm, try to put forward ideas from a relatively novel perspective
2. Optimization of model parameters, try to use different functions
3. Operation optimization, try to use different operation operators and parameters
4. Try not to overfit the parameters
5. Try not to use too many conditional logic judgments



## 7.2 Some idea sources

1. Price trends and reversals
2. Volatility analysis
3. Combination of price and volume
4. Combination of long-term and short-term trends
5. Seek inspiration from various websites and literature



> **Note**: The above discussion is meant to give in a simple idea of how important is programming for executing the theoretical matter taught to students. This does not mean the reader will leave the page having a clear idea of what goes on actually in a quant workshop, but the reader will now be sure of what he/she does not know.
