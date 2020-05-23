---
layout: post
title: Asset Allocation with VAR and LSTM
---
## Introduction
I have seen numerous attempts on Medium applying crude neural networks, 
LSTM models on predicting a single stock's price only using last day's price. 
While most underperforming a simple ARIMA model and authors total diregarding stationarity, normality, trends, correlations etc. we have seen some billiant price returns prediction using ML approaches on the academic side, [paper](https://dachxiu.chicagobooth.edu/download/ML.pdf)
and using that to construct a portfolio that outperforms traditional Markowitz's MVO framework [paper](http://www.thinkmind.org/download.php?articleid=intsys_v11_n12_2018_3) 
[paper](https://www.researchgate.net/profile/Lasse_Lundqvist/publication/338712365_A_Deep_Learning_Model_in_a_Tactical_Asset_Allocation_Framework_Lasse_Lundqvist/links/5e26d670299bf1031e27d154/A-Deep-Learning-Model-in-a-Tactical-Asset-Allocation-Framework-Lasse-Lundqvist.pdf). 


These give me a string of ideas, 
=> LSTM can modelled multivariate time series so why just predicting one stock, one asset?
=> as a potfolio manager we want to model several assets future performance to give a robust portfolio
=> VAR, Vector Autoregressive Model would be the econoemtrics approach, which essentially a matrix form of ARIMA across varaibles
=> can we use a LSTM in predicting cross-market returns that outperforms a VAR?
=> using the same LSTM model to construct a portfolio that outperforms a VAR? 

### Hypothesis
1. a LSTM model that predicts returns of the markets, outperforming VAR
2. using the predicted returns, we can form a portfolio that outperforms the classic Markowitz's MVO portfolio

#### Why LSTM maybe useful?  
The argument of using LSTM in time series forecasting is LSTM cell remembers the past states while processing sequence of data in contrast to a feedforward neural network. 
That being said, it seems reasonable to assume you would need some sort of linear/ non-linear relationships between the sequence and your target, the sequence of past returns and the forward looking return in our case.

#### Why combining VAR and LSTM
I read this [paper](https://goelhardik.github.io/images/Multivariate_Aviation_Time_Series_Modeling_VARs_vs_LSTMs.pdf) and this [tutorial](https://towardsdatascience.com/combine-lstm-and-var-for-multivariate-time-series-forecasting-abdcb3c7939b). The first study shows a simple VAR outperforms LSTM while the second tutorial shows by feeding VAR predictions to the LSTM model, the MAR of all predicting variables are marginally better. 

#### Data
To keep things simple for now, we are using US, UK, Japan, Germany, EM of equity and bond total return indices spanning from 1990-2019. We splitted the data to train , validation and test set. We can add more features such as market sentiment, VIX, momentum later on.

#### 1.1 Building the VAR 
First find the optimal lag by in-sample fitting the train set. We will use the model with the lowest AIC to produce VAR predictions. 
#### 1.2 Building the LSTM
Like using most ML methods, we'll use a Standard Scalar to normalised the data. As mentioned, we can feed a sequence to produce n-step predictions. Kera's (Timeseries Generator)[https://keras.io/api/preprocessing/timeseries/#timeseriesgenerator-class] allows us to map sequence to the target. e.g.
```
 TimeseriesGenerator([1,2,3,4,5,6,7,8],[1,2,3,4,5,6,7,8], length=5) gives 
 [1,2,3,4,5] =>[6]
 [2,3,4,5,6] => [7] ...
```







