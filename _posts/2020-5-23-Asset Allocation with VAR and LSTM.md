---
layout: post
title: Asset Allocation with VAR and LSTM
published: true
---
## Introduction
Recently I read this [paper](http://www.thinkmind.org/download.php?articleid=intsys_v11_n12_2018_3). The authors show they built a portfolio that outperformed a MVO portfolio using predicted returns from a LSTM model. I was a bit sceptical, not only because I have seen numerous fail attempts in predicting a single stock's price using LSTM models (most underperform a simple ARIMA model). I think the LSTM's predictions of forward returns are essentially past returns, which stabilise the weights of the portfolio, giving a better portfolio than the classic Markowitz's MVO portfolio.


### Hypothesis
1. a LSTM model that predicts returns of the markets, outperforms a Vector Autoregressive (VAR) model
2. a portfolio that outperforms the classic Markowitz's MVO portfolio even if the LSTM's predictions underperforms a VAR. And a portfolio using VAR predictions will also outperform the classic Markowitz's MVO portfolio

#### Why LSTM maybe useful?  
The argument of using LSTM in sequence forecasting is LSTM cell remembers the past states while processing incoming sequence of data in contrast to a feedforward neural network. 
That being said, it seems reasonable to assume you would need some sort of linear/ non-linear relationships between the sequence and your target, the sequence of past returns and the forward looking returns in our case.

I have also read this [paper](https://goelhardik.github.io/images/Multivariate_Aviation_Time_Series_Modeling_VARs_vs_LSTMs.pdf) and this [tutorial](https://towardsdatascience.com/combine-lstm-and-var-for-multivariate-time-series-forecasting-abdcb3c7939b). The first study shows a simple VAR outperforms LSTM while the second tutorial shows by feeding VAR predictions to the LSTM model, the MAE of all predicting variables are marginally better. I'll incorporate VAR predictions as additional features for a second model, VAR-LSTM to see if there's any improvement.
#### Data
To keep things simple for now, I am using US, UK, Japan, Germany, EM equity and bond total return indices spanning from 1990-2019. I split the data to train, validation and test set. More features are added such as market sentiment, VIX, momentum etc. for model LSTM_X.  
#### 1.1 Building the VAR 
The optimal lag is found by in-sample fitting the train set. The model with the lowest AIC is used to produce VAR predictions.

#### 1.2 Preparing time series generator for LSTM
Like with most ML methods, I use a Standard Scalar to normalised the data. Stationarity in LSTM is not absolutely necessary as it doesn't have the same assumption as VAR. But it is more effective as the differenced features/ target should share same, at least similar distributions across train, validation and test set. One method suggested by Marcos Lopez de Prado is fractionally differentiated features which keeps the memory as much as possible. Some interesting papers deal with non-stationarity and differential signals that I want to try my hands on in future posts [paper](https://papers.nips.cc/paper/8672-shape-and-time-distortion-loss-for-training-deep-time-series-forecasting-models.pdf) [paper](https://arxiv.org/pdf/1811.07490.pdf).
As mentioned, we can feed a sequence to produce n-step predictions. Kera's (Timeseries Generator)[https://keras.io/api/preprocessing/timeseries/#timeseriesgenerator-class] allows me to map sequence to the target. e.g.
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
We'll build several models fitted with different features. 
Features variation: 
 - only past price returns are fed as features (LSTM) 
 - additional features such as bond yields, market sentiments, price ratios etc. (LSTM_X)
 - past returns and VAR predictions sequence (VAR_LSTM)

#### 1.4 Results
We used the R2 and MAE to compare 1. VAR 2. LSTM 3. LSTM_X 4. VAR_LSTM 

![r2_1_X_stacked]({{site.baseurl}}/images/r2_1_X_stacked.png){: height="400px" width="auto"} 
![MAE_1_X_stacked]({{site.baseurl}}/images/MAE_1_X_stacked.png){: height="450px" width="auto"} 




 Surprisingly, both LSTM, LSTM_X and VAR_LSTM outperforms VAR in UK Equity and Germany Equity price returns while underperforms in UK Bond, Japan Bond and EM Equity. But with scoring negative R2 in other markets and sharing similar MAEs across models, I won't say any of the models are great. Do any of the models really predict price returns? If we translate returns back to price levels, we'll see all models' predictions merely lagging the real price level, very similar to the VAR model. 

![price_US_Equity_X]({{site.baseurl}}/images/price_US_Equity_X.png){: height="400px" width="auto"} 
![price_UK_Equity_X]({{site.baseurl}}/images/price_UK_Equity_X.png){: height="400px" width="auto"} 
![price_Japan_Equity_X]({{site.baseurl}}/images/price_Japan_Equity_X.png){: height="400px" width="auto"} 
![price_US_Bond_X]({{site.baseurl}}/images/price_US_Bond_X.png){: height="400px" width="auto"} 


What if we bin the returns to buckets of [-1, 0, 1] and check the confusion matrix? If we look at all markets predictions of each model, every model seems randomly guessing if the markets are going up or down.

![all_assets_VAR]({{site.baseurl}}/images/all_assets_VAR.png){: height="50px" width="auto"} 
![all_assets_LSTM]({{site.baseurl}}/images/all_assets_LSTM.png){: height="50px" width="auto"} 
![all_assets_LSTM_X]({{site.baseurl}}/images/all_assets_LSTM_X.png){: height="50px" width="auto"} 
![all_assets_VAR_LSTM]({{site.baseurl}}/images/all_assets_VAR_LSTM.png){: height="50px" width="auto"} 


If we dive down into individual asset, we see Japan Equity and Bond are doing marginally better than other markets, but performances are similar across all models, including the simple VAR.

![VAR_US%20Equity]({{site.baseurl}}/images/VAR_US%20Equity.png){: height="50px" width="auto"} 
![VAR_US%20Bond]({{site.baseurl}}/images/VAR_US%20Bond.png){: height="50px" width="auto"} 
![VAR_Japan%20Equity]({{site.baseurl}}/images/VAR_Japan%20Equity.png){: height="50px" width="auto"} 
![VAR_Japan%20Bond]({{site.baseurl}}/images/VAR_Japan%20Bond){: height="50px" width="auto"} 
![LSTM_X_Japan%20Bond]({{site.baseurl}}/images/LSTM_X_Japan%20Bond.png){: height="50px" width="auto"} 
![LSTM_X_Japan%20Equity]({{site.baseurl}}/images/LSTM_X_Japan%20Equity.png){: height="400px" width="auto"} 
![LSTM_X_US%20Bond]({{site.baseurl}}/images/LSTM_X_US%20Bond.png){: height="50px" width="auto"} 
![LSTM_X_US%20Equity]({{site.baseurl}}/images/LSTM_X_US%20Equity.png){: height="50px" width="auto"} 



So there you have it, **LSTM performs no better than VAR in predicting forward return**. 

Could it be the models have different features compared to the study? Maybe, but we'll never know unless the authors reveal the identities of the 387 features (which are deduced to 115 principal components). I have also tried monthly returns, which is the frequency used in the study. Even though LSTM_X is the only model with positive R2 in UK Equity, Japan Equity, UK Bond, Germany Bond and Canada Bond. But the out of sample performance is worse than the daily frequency for most markets. 
![r2_20_X_stacked]({{site.baseurl}}/images/r2_20_X_stacked.png){: height="50px" width="auto"} 
![MAE_20_X_stacked]({{site.baseurl}}/images/MAE_20_X_stacked.png){: height="50px" width="auto"} 

Could the result be better if I adjust target returns by realised volatility or even smooth the features or the target with a tanh function to adjust for low noise-signal ratio? Possibly, but that would be another post. 

Going back to our second hypothesis, I want to know if these poor predictions give a better portfolio than MVO. My guess would be not, as it simply updates the VCV matrix, weights with the latest returns. The below index portfolios are rebalanced monthly.
![port_plot]({{site.baseurl}}/images/port_plot.png){: height="600px" width="auto"} 

As expected, quite similar. Inspecting the annualised Sharpe and mean Sharpe.


|      |    MVO |    VAR |   LSTM |   LSTM_X |   VAR_LSTM |
|------|--------|--------|--------|----------|------------|
| 2016 |  -1.13 |  -1.11 |  -1.15 |    -1.12 |      -1.15 |
| 2017 |   2.7  |   2.74 |   2.74 |     2.64 |       2.71 |
| 2018 |   1.5  |   1.4  |   1.35 |     1.23 |       1.34 |
| 2019 |   0.19 |   0.45 |   0.44 |     0.32 |       0.51 |


|              |   MVO |   VAR |   LSTM |   LSTM_X |   VAR_LSTM |
|--------------|-------|-------|--------|----------|------------|
|  mean Sharpe |  0.82 |  0.87 |   0.85 |     0.77 |       0.85 |


Suprisingly VAR slightly outperforms all other portfolios. They all share similar performances. And all have better Sharpe than the portfolios in the study.

**LSTM portfolios show only slight performance enhancement from MVO, but underperforms VAR.**

Is there anything we can do better? There's a landmark paper from Bryan Kelly, AQR [paper](https://dachxiu.chicagobooth.edu/download/ML.pdf) that construct different portfolios with a bigger investment universe(US individual stocks), a broader ML approaches (xgb, deep and shallow neural networks) and more technical analysis features (top features ranked by importance are 1-month momentum, 1-month max return ...). I have tried to replicate the paper and shows similar results.

Stay Tuned. 

Welcome for any feedback and discussion!
