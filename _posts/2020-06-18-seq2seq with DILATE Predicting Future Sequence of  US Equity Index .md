---
published: true
---
## DILATE
** full code [repo](https://github.com/kingwongf/financeDILATE/blob/master/README.md) ** 

I recently read the [paper](https://github.com/vincent-leguen/DILATE), "Shape and Time Distortion Loss for Training Deep Time Series Forecasting Models" published at NEURIPS 2019. It gives a fresh approach to the common problem of non-stationarity in time seies forecasting. The authors introduced 
DILATE (DIstortion Loss including shApe and TimE) as a new objective loss function. The loss is composed of 1. Shape loss 2. Temporal loss. 
The shape loss is based on Dynamic Time Warping, which measures the structural similarity between the prediction and targets by temporarily aliging the two sequences. Using [Soft-DTW](http://proceedings.mlr.press/v70/cuturi17a/cuturi17a.pdf), the non-differerntiable DTW becomes differntiable by transforming it via smooth-min. This allows the model to update params as the loss propagates backward.

The temporal loss is based on Time Distortion Index, which is the deviation between the DTW optimal path and the first diagonal pairwise distances between each points in two set of sequences. The authors rewrote TDI into the arg min of the optimal DTW path and Omega, a squared penalising matrix of each point in the sequences of targets and predictions. Since the loss term is still non-differerntiable, the author defined a smooth approximation of the arg min. 

The authors tested their approach with a seq2seq model (a RNN encoder combined with a RNN decoder). The reason to choose seq2seq is that the loss term must be computed on the target and prediction sequences instead of predicting a single/ multiple point estimates. They tested on 1. random simulated data which consists of two spikes 2. ECG data 3. Traffic data. The results are promising as the authoer's model outperformed or equivalent in terms of DTW, TDI and MSE losses when compare to models using MSE or DTW as loss function.

I decided to get my hands dirty by trying to some forecast financial time series (typical...). I forked the paper's [repo](https://github.com/vincent-leguen/DILATE), which only consists of an univariate time series simulation forecasting example. 

## Motivation
predicting future price pattern using historical price patterns is useful as we can bet of future price movement, model future volatility etc. 

## Goal
to forecast the total return  US Equity index, and other market indices next 3 day movements based on past 1 month sequences of other regions markets indices, other asset prices, price ratios, factors etc. We can extend the input and output sequence to 60 and 5 for further development.

## Methodology
First I used the purged k fold techniques to split the time series data into batches of train and test sets. The train and test sets are further spltted into sequences of features and targets. Since this is a seq2seq model, I can just split the first 20 observations into one array (20, n_features) of inputs and the last 3 observations into one array (3,1) of target for each window. 

To convert the univariate seq2seq to a multivariate case I simply reshaped the inputing tensor layers in the encoder and the hidden layer used by decoder, the decoder however will still predicts one sequence as we only want to predict one sequence at a time. 

Then I seemed to hit a small problem, scaling. ML models usually work better when you standardised you inputs. However, time series data can't be simply scaled by the Standard Scalar due to a definite leakage from the full data mean and standard deviation. Another reason not to scale the data is that the authors purposed model DILATE is aimed to solve non-stationarity forecasting. It supposed to work with inputs and targets with time-variant moments. When training the model with raw data, unsurprisingly the optimisation quickly leads to divergence/ loss: nan. So, I have tried to natural log the raw data instead. But the optimisation also stopped earlier than intended as the actual loss is much smaller due to the natural log. 
There are few options I tried in the end, 1. dynamically scale each data sample by train input's means and standard deviation (like a rolling window) 2. dynamically scale each day data by an expanding window of means and standard deviation 3. use log transformation, but train very slowly with a batch size of one hoping the model converges to a small enough loss that won't explode exponentially after inverse transform back to its original scale.

My hypothesis is the predictions will different froma a typical time series forecast model using MSE loss. Instead of giving lagged value as prediction in a MSE loss model, the DILATE model should give a sequence prediction that matches typical price patterns such as trending upward and downward or even channel or breakout patterns. 


Three seq2seq models with the same architecture but three different losses are trained and tested out-of-sample 1. MSE Loss 2. Soft DTW 3. DILATE

## Results
The results are a bit surprising as DILATE did not score the smallest out-of-sample loss in both DTW and TDI like in the original paper across all scaling methods. The DILATE model only outperformed in in terms of TDI and scored marginally bottom in terms of DTW and MSE. 

![to_post]({{site.baseurl}}/images/dilate_results_2.png){: height="250px" width="auto"}


But there is still an argument of using loss objective function such as Soft-DTW or DILATE other than MAE or MSE when dealing with time-series data. Below is one of the out-of-sample testings. We see Soft-DTW prediction has the best fit, more importantly, DILATE and Soft-DTW outperformed MSE as they correctly predicted price trended upward and dropped as oppose to MSE predicting a fall and a rise later. 

![to_post]({{site.baseurl}}/images/to_post.png){: height="450px" width="auto"} 

But of course we also see when all three models got the prediction wrong in various degrees and MSE loss model has the smallest losses.

![to_post]({{site.baseurl}}/images/1195.png){: height="250px" width="auto"}


When comparing results of different scaling methods using MSE GRU net's losses as the benchmark, we see DILATE has the largest reductions in losses.

![to_post]({{site.baseurl}}/images/dtw_dilate.png){: height="450px" width="auto"} 

The combination of Net DILATE with the exapnding Standard Scalar method has the largest reduction in MSE and DTW losses (inf in TDI due to dividing by zero, a 0 to 0.69 increase). 

## Next steps
So I have a seq2seq net that outperforms a seq2seq using MSE in predicting the US Equity Total Return Index. Next would be running explanatory analysis such as [SHAP](https://github.com/slundberg/shap) and try predicting different time series. I'll leave that to you [repo](https://github.com/kingwongf/financeDILATE/blob/master/README.md)
