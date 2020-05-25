---
layout: post
title: Asset Allocation with VAR and LSTM
published: true
---
## Introduction
Recently I read this [paper](http://www.thinkmind.org/download.php?articleid=intsys_v11_n12_2018_3). Tha authors show they built a portfolio that outperformed a MVO portfolio using predicted returns from a LSTM model. I was a bit skeptical, not only because I have seen numerous fail attempts in predicting a single stock's price using LSTM models (most underperform a simple ARIMA model), but I think the LSTM's prediction of daily returns are essentially stabilising the weights of the portfolio, resulting a better portfolio than the classic Markowitz's MVO portfolio.



### Hypothesis
1. a LSTM model that predicts returns of the markets, outperforming VAR
2. a portfolio that outperforms the classic Markowitz's MVO portfolio even if the LSTM's predictions underperforms a VAR. And a portfolio using VAR predicitions will also outperforms the classic Markowitz's MVO portfolio

#### Why LSTM maybe useful?  
The argument of using LSTM in time series forecasting is LSTM cell remembers the past states while processing sequence of data in contrast to a feedforward neural network. 
That being said, it seems reasonable to assume you would need some sort of linear/ non-linear relationships between the sequence and your target, the sequence of past returns and the forward looking return in our case.

I have also read this [paper](https://goelhardik.github.io/images/Multivariate_Aviation_Time_Series_Modeling_VARs_vs_LSTMs.pdf) and this [tutorial](https://towardsdatascience.com/combine-lstm-and-var-for-multivariate-time-series-forecasting-abdcb3c7939b). The first study shows a simple VAR outperforms LSTM while the second tutorial shows by feeding VAR predictions to the LSTM model, the MAE of all predicting variables are marginally better. I'll incorporate VAR predictions as additional features for a second model, VAR-LSTM to see if there's any improvement.
#### Data
To keep things simple for now, I using US, UK, Japan, Germany, EM of equity and bond total return indices spanning from 1990-2019. We splitted the data to train , validation and test set. We add more features such as market sentiment, VIX, momentum etc. later on.

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
We used the R2 and MAR to compare 1. VAR 2. LSTM, uses only past returns as features 3. LSTM_X uses additional features such as market sentiment, momentum, price-ratios 4. VAR_LSTM uses VAR predictions and past returns. 

<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/r2_1_X_stacked.png" width="400" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/MAE_1_X_stacked.png" width="400" alt="hi" class="inline"/>



 Surprisingly, both LSTM, LSTM_X and VAR_LSTM outperforms VAR in UK Equity and Germany Equity price returns while underpforms in UK Bond, Japan Bond and EM Equity. But with scoring negative R2 in other markets and sharing similar MAEs across models, I won't say any of the models are great. Do any of the models actually predict price returns? If we translate returns back to price levels, we'll see all models' predictions merely lagging the real price level, very similar to VAR/ ARIMA model. 


<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/price_US_Equity_X.png" width="400" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/price_UK_Equity_X.png" width="400" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/price_Japan_Equity_X.png" width="400" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/price_US_Bond_X.png" width="400" alt="hi" class="inline"/>

What if we bin the returns to buckets of [-1, 0, 1] and check the confusion matrix? If we look at all markets predicions of each model, every model seems randomly guessing if market's going up or down.

<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/all_assets_VAR.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/all_assets_LSTM.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/all_assets_LSTM_X.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/all_assets_VAR_LSTM.png" width="100" alt="hi" class="inline"/>
If we dive down into individual asset, we see Japan Equity and Bond are doing marginally better than other markets, but performances are similar across all models, including the simple VAR.

<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/VAR_US%20Equity.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/VAR_US%20Bond.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/VAR_Japan%20Equity.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/VAR_Japan%20Bond.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/LSTM_X_Japan%20Bond.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/LSTM_X_Japan%20Equity.png" width="100" alt="hi" class="inline"/>

<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/LSTM_X_US%20Bond.png" width="100" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/LSTM_X_US%20Equity.png" width="100" alt="hi" class="inline"/>


So there you have it, **LSTM performs no better than VAR in predicting forward return**. 

Could it be we have different features compare to the study? Maybe, but we'll never know unless the authors reveal the identities of the 387 features (which are deduced to 115 principle compenents)  I have also tried monthly returns, which is the frequency used in the study. Even though LSTM_X is the only model with positive R2 in UK Equity, Japan Equity, UK Bond, Germany Bond and Canada Bond. But the out of sample performance are actually worse than daily frequency for most markets. 
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/r2_20_X_stacked.png" width="400" alt="hi" class="inline"/>
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/MAE_20_X_stacked.png" width="400" alt="hi" class="inline"/>

Could the result be better if we adjust target returns by realised volatility or even smooth the features or the target with a tanh function to adjust for low noise-signal ratio? Possibily, but that would be another series of posts. 
Going back to our second hypthesis, I want to know if these poor predictions actually give a better portfolio than MVO. My guess would be not, as it simply updates the VCV matrix, weights with the latest returns. The below index portfolios are rebalanced monthly.
<img src="https://github.com/kingwongf/kingwongf.github.io/blob/master/images/port_plot.png" width="600" alt="hi" class="inline"/>
As expected, quite similar. Inspecting the annualised Sharpe and mean Sharpe.

|      |    MVO |    VAR |   LSTM |   LSTM_X |   VAR_LSTM |
|------|--------|--------|--------|----------|------------|
| 2016 |  -1.13 |  -1.11 |  -1.15 |    -1.12 |      -1.15 |
| 2017 |   2.7  |   2.74 |   2.74 |     2.64 |       2.71 |
| 2018 |   1.5  |   1.4  |   1.35 |     1.23 |       1.34 |
| 2019 |   0.19 |   0.45 |   0.44 |     0.32 |       0.51 |


|     		   |   MVO |   VAR |   LSTM |   LSTM_X |   VAR_LSTM |
|--------------|-------|-------|--------|----------|------------|
|  mean Sharpe |  0.82 |  0.87 |   0.85 |     0.77 |       0.85 |


Suprisingly VAR slightly outperforms all other portfolios. They are share similar performances. 

**LSTM portfolios show only slight performance enhancement from MVO, but underperforms VAR.**

Is there anything we can do better? There's a landmark paper from Bryan Kelly, AQR [paper](https://dachxiu.chicagobooth.edu/download/ML.pdf) that construct portfolio with a bigger universe(US individual stocks), a broader ML approaches (xgb, deep and shallow neural networks) and more technical analysis features (top features are 1 month momentum, 1 month max return ...). I have tried to replicate the paper and shows similar results. 
Stay Tuned. 



