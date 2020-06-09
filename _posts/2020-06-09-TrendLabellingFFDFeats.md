---
published: false
---

## Trend Labelling, Volatility Adjusted returns or Raw Forward Returns? Fractionally Differenced Log Prices, Log Prices or Raw Prices or even Wavelet Transformed?

I always face a dilemma when building predicitve model. 
What target should I use? Marcos Lopez de Prado purposed few labelling methods such as the triple-barrier: using predfined stop-loss, profit taking level and holding period to label the data, trend following label: arg max of a look-forward period of t-vals by running ols on price trend as opposed to the common practices of volatility adjusted returns or a smooth version of the noisy forward returns. Some may even turn the problem into a classifier problem, but this also disregard the magnitude of error (10% and 1% returns are very different but both fall into bin: +1/ long). 

What features should I use? There are so many differnt features claim to be successful out there. Fractionally differenced and entropy based features purposed by Marcos Lopez de Prado again in his books "Advances in Financial Machine Learning" and "Machine Learning for Asset Managers" respectively. Head of ML, AQR Bryan Kelly also published a detailed list of features that claims to work on predicting monthly returns. [paper](https://dachxiu.chicagobooth.edu/download/ML.pdf) 

I decided to try all labellings and features combinations, also adding the features from the paper as close as posssible. I am using the historical SP500 EOD , OHLCV with dividened data spanning from 1965 to 2019. I also have a mapping of ticker to sectors to calculate sector related features used in the paper. Some features are missing in my implementation such as bid-ask spread, valuation ratios and fundamental signals due to lack of data (if you can provide the necessary data from CRSP, I would be very happy). 

The reason to use fractionally differenced log prices is to obtain a stationary time series while maintain the memory process of the log prices. Instead of an interger differnecing (e.g. calclating returns), where stationarity is guranteed in the price of losing persistent pattern of non-zero correlations that is between lag 0 and 1, the time series is fractionally differenced to minimise information loss. I sampled the code from "Advances in Financial Machine Learning" for this project. Optimizing the differencing threshold using the train dataset is hell of a job due to the recrusive run of the ADF test. Instead of calling for the optimization of sklearn as suggested in the book, I simply run a fine grid of the possible thresholds and choose the one which has the lowest cost. And to parallerize the operation, I used the package [Ray](https://github.com/ray-project/ray). Ray is an amazing package when you need parallelisation and it even has a hyperparameter tuning package called [Tune](https://docs.ray.io/en/latest/tune.html). I will write a seperate post on how I used it for pandas operation. 

The other features combinations are log prices and raw prices. I'll leave entropy-based features and Harr-Wavelet transformation in the next post. 

Trend labelling is simple and intuitive. The t-value of the time trend to price level is calculated for at each point in time for a look-ahead period. An expanding window within the look-ahead period is used so the one with the largest absolute t-val is used. Again I used Ray to parallerize the whole process. To calculate volatility-adjusted forward return, the forward returns are simply divided by the rolling volatility. 





Due to the shear size of the data, I calculated price trends features such as momentum with 