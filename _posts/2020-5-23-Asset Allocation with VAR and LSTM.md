---
layout: post
title: Asset Allocation with VAR and LSTM
published: true
---
## Introduction
Recently I read this [paper](http://www.thinkmind.org/download.php?articleid=intsys_v11_n12_2018_3). Tha authors show they built a portfolio that outperformed a MVO portfolio using predicted returns from a LSTM model. I was a bit skeptical, not only because I have seen numerous fail attempts in predicting a single stock's price using LSTM models (most underperform a simple ARIMA model), but I think the LSTM's prediction of daily returns are essentially stabilising the weights of the portfolio, resulting a better portfolio than the classic Markowitz's MVO portfolio.



### Hypothesis
1. a LSTM model that predicts returns of the markets, underperforming VAR
2. we can form a portfolio that outperforms the classic Markowitz's MVO portfolio even if the LSTM's predictions underperforms a VAR. And a portfolio using VAR predicitions will also outperforms the classic Markowitz's MVO portfolio

#### Why LSTM maybe useful?  
The argument of using LSTM in time series forecasting is LSTM cell remembers the past states while processing sequence of data in contrast to a feedforward neural network. 
That being said, it seems reasonable to assume you would need some sort of linear/ non-linear relationships between the sequence and your target, the sequence of past returns and the forward looking return in our case.

I have also read this [paper](https://goelhardik.github.io/images/Multivariate_Aviation_Time_Series_Modeling_VARs_vs_LSTMs.pdf) and this [tutorial](https://towardsdatascience.com/combine-lstm-and-var-for-multivariate-time-series-forecasting-abdcb3c7939b). The first study shows a simple VAR outperforms LSTM while the second tutorial shows by feeding VAR predictions to the LSTM model, the MAE of all predicting variables are marginally better. I'll incorporate VAR predictions as additional features for a second model, VAR-LSTM to see if there's any improvement.
#### Data
To keep things simple for now, we are using US, UK, Japan, Germany, EM of equity and bond total return indices spanning from 1990-2019. We splitted the data to train , validation and test set. We add more features such as market sentiment, VIX, momentum etc. later on.

#### 1.1 Building the VAR 
First find the optimal lag by in-sample fitting the train set. We will use the model with the lowest AIC to produce VAR predictions. 
#### 1.2 Preparing time series generator for LSTM
Like using most ML methods, we'll use a Standard Scalar to normalised the data. As mentioned, we can feed a sequence to produce n-step predictions. Kera's (Timeseries Generator)[https://keras.io/api/preprocessing/timeseries/#timeseriesgenerator-class] allows us to map sequence to the target. e.g.
```
 features = [[11,12,13,14,15,16,17,18],
             [21,22,23,24,25,26,27,28]]
 target = [1,2,3,4,5,6,7,8]
 TimeseriesGenerator(features, target, length=5) gives 
 [[11,12,13,14,15],
  [21,22,23,24,25]] to [6]
 [[12,13,14,15,16],
  [22,23,24,25,26]] to [7] ...
```


#### 1.3 Building LSTMs
We'll build several models fitted with differnt features and architecture. 
Features variation: only price returns are fed as features vs. other features such as bond yields, market sentiments, price ratios etc.
Architecture variation: single LSTM vs staked LSTM, a stacked LSTM as deeper couls be better [paper](https://www.jair.org/index.php/jair/article/view/11030/26198)

#### 1.4 Results
We first look at the model trained only with past returns, which we expect it performs as good as a VAR. 
![r2_1_stacked.png]({{site.baseurl}}/images/r2_1_stacked.png)
![MAE_1_stacked.png]({{site.baseurl}}/images/MAE_1_stacked.png)



 Surprisingly, both LSTM and VAR_LSTM outperforms in predicting US Equity, UK Equity, Germany Equity, Canada Equity price returns compare to the VAR model while underpforms in Japan Equity, UK Bond, Japan Bond and EM Equity. But do any of the models actually predict price returns? If we translate returns back to price levels, we'll see all models predictions merely lagging the real price level, . 



<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/price_Japan_Equity.png" width="48">





