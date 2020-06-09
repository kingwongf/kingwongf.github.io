---
published: false
---

## Trend Labelling, Volatility Adjusted returns or Raw Forward Returns? Fractionally Differenced Log Prices, Log Prices or Raw Prices or even Wavelet Transformed?

I always face a dilemma when building predicitve model. 
What target should I use? Marcos Lopez de Prado purposed few labelling methods such as the triple-barrier: using predfined stop-loss, profit taking level and holding period to label the data, trend following label: arg max of a look-forward period of t-vals by running ols on price trend as opposed to the common practices of volatility adjusted returns or a smooth version of the noisy forward returns. Some may even turn the problem into a classifier problem, but this also disregard the magnitude of error (10% and 1% returns are very different but both fall into bin: +1/ long). 

What features should I use? There are so many differnt features claim to be successful out there. Fractionally differenced and entropy based features purposed by Marcos Lopez de Prado again in his books Advances in Financial Machine Learning and Machine Learning for Asset Managers respectively. 