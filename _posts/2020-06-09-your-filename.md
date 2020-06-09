---
published: false
---
## DILATE

I recently read the [paper](https://github.com/vincent-leguen/DILATE)er, "Shape and Time Distortion Loss for Training Deep Time Series Forecasting Models" published at NEURIPS 2019. It gives a fresh approach to the common problem of non-stationarity in time seies forecasting. The authors introduced 
DILATE (DIstortion Loss including shApe and TimE) as a new objective loss function. The loss is composed of 1. Shape loss 2. Temporal loss. 
The shape loss is based on Dynamic Time Warping, which measures the structural similarity between the prediction and targets by temporarily aliging the two sequences. Using [Soft-DTW](http://proceedings.mlr.press/v70/cuturi17a/cuturi17a.pdf), the non-differerntiable DTW becomes differntiable by transforming it via smooth-min. This allows the model to update params as the loss propagates backward.
The temporal loss is based on Time Distortion Index, which is the deviation between the DTW optimal path and the first diagonal pairwise distances between each points in two set of sequences. The authors rewrote TDI into the arg min of the optimal DTW path and Omega, a squared penalising matrix of each point in the sequences of targets and predictions. Since the loss term is still non-differerntiable, the author defined a smooth approximation of the arg min. 

The authors tested their approach with a seq2seq model (a RNN encoder combined with a RNN decoder). The reason to choose seq2seq is that the loss term must be computed on the target and prediction sequences instead of predicting a single/ multiple point estimates. They tested on 1. random simulated data which consists of two spikes 2. ECG data 3. Traffic data. The results are promising as the authoer's model outperformed or equivalent in terms of DTW, TDI and MSE losses when compare to models using MSE or DTW as loss function.

I decided to get my hands dirty by trying to some forecast financial time series (typical...). I forked the paper's [repo](https://github.com/vincent-leguen/DILATE), which only consists of an univariate time series simulation forecasting example. 
Goal: try to forecast US Equity index, and other market indices next 3 day movements based on past 1 month sequences of other regions markets indices, other asset prices, price ratios, factors etc. 

First I used the purged k fold techniques to split the time series data into batches of train and test sets. The train and test sets are further spltted into sequences of features and targets. Since this is a seq2seq model, I can just split the first 20 observations into one array (20, n_features) of inputs and the last 3 observations into one array (3,1) of target for each window. 

To convert the univariate seq2seq to a multivariate case I simply reshaped the inputing tensor layers in the encoder and the hidden layer used by decoder, the decoder however will still predicts one sequence as we only want to predict one sequence at a time. 

Then I seemed to hit a small problem, scaling. Time series data can't be just scaled by Standard Scalar due to a definite leakage from the data mean and standard deviation. I tried to natural log the raw data so the loss wouldn't diverge to infinity/ nan, but the optimiaation also stopped earlier than intended as the actual loss is much smaller due to the natural log. There are few options I tried in the end, 1. dynamically scale each data sample by train input's means and standard deviation (like a rolling window) 2. dynamically scale each day data by an expanding window 3. use log transformation 

