---
layout: post
title: Asset Allocation with VAR and LSTM
---
## Introduction
I have seen numerous attempts on Medium applying crude neural networks, 
LSTM models on predicting a single stock's price only using last day's price. 
Most fail, underperforming a simple ARIMA model, since the authors really thought yesterday's price tells you anything about forward looking returns and 
total diregarding stationarity, normality, trends, correlations etc.
On the academic side however we have seen some billiant price returns prediction using ML approaches, [paper](https://dachxiu.chicagobooth.edu/download/ML.pdf)
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
That being said, it seems reasonable to assume you would need some sort 
####

