---
title: "Portfolio Optimization with Deep Learning"
classes: wide
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
---

### Summary
We focus on applying deep learning to the problem of portfolio optimization in long-only strategies and compare their results to mean-variance portfolio optimization baselines. 

For a set of n assets and their historical prices, we aim to find a optimal set of n weights such that their Sharpe ratio is maximized over a future horizon. This problem has been explored thoroughly in existing literature, starting from the Markowitz model in 1952. Mean-variance optimization uses inputs of n expected returns and an nxn covariance matrix. Modern portfolio theory have explored different approaches but generally approach the problem by creating assumptions on the return profiles of the assets, then using an optimization method to arrive at risk-return optimized weights. 

We want to explore an integrated optimization method instead, using inputs of historical asset prices and directly learning the optimal weights, instead of having an intermediate return prediction step. We first replicate the results of an existing paper from a team at Oxford(Zhang, 2005), then we attempt to address problems of the existing model by introducing exogenous features and different deep learning architectures.

<div style="text-align: center;">
  <img src="/assets/images/port_opt/problem_statement.png" alt="Problem Statement" width="750"/>
  <figcaption>Problem Diagram</figcaption>
</div>

### Theory

<div style="text-align: center;">
  <img src="/assets/images/port_opt/zhang_architecture.png" alt="Zhang Architecture" width="500"/>
  <figcaption>Zhang Architecture</figcaption>
</div>

The Zhang paper introduced the use of LSTM for feature learning and prediction with the use of a Sharpe ratio loss function for backpropagation. The loss is calculated as 

$$
\text{Sharpe Loss} = \frac{R_p - R_f}{\sigma_p}
$$

The optimization strategy maximizes the Sharpe ratio via a neural network, bypassing traditional forecasting by linking asset returns and prices over a trading period. It ensures positive weights sum to one using a SoftMax function. Gradient ascent optimizes the framework during training, updating parameters iteratively for Sharpe ratio convergence. Zhang's LSTM architecture includes layers for input, LSTM, and SoftMax output, with mini-batches spanning n days and using observations from the past m days to optimize daily asset weights for the Sharpe ratio. Features are extracted from an n-day look-back window, linked to the next day's asset returns.

During training, Zhang et al. used a validation set comprising 10% of the training data. They identified the most effective model with hyperparameter tuning of LSTM with 64 units, a 50-day lookback window, and a mini-batch size of 64, achieving convergence within 100 epochs. During training, we found that a learning rate of 0.001 led to optimal model convergence and performance during training. We assumed a zero risk-free rate.

Zhang's training process spanned from 2006 to the end of 2010, with periodic retraining every two years using all available data up to that point to update parameters. Their testing period extended from 2011 to the end of April 2020, providing a comprehensive assessment of their model's performance over a significant timeframe. 

### Methods
#### Data
Our data consists of four adjusted time series of stocks (VTI), bonds (AGG), commodities (DBC) and volatility (VIX Index) from 02/07/2006 to 04/30/2020 obtained from Yahoo Finance API. A series of exogenous variables were obtained for use in the exogenous data enhancement. Although the VIX Index is not a directly tradeable asset unlike the total return series of the other ETFs, we included it since it was used in the paper from Zhang et al. and sought to replicate their values. 

#### Replication
Using the methods described above, we were able to replicate the results seen in the Zhang paper. We saw rolling Sharpe performance consistently stronger than the MVO and equal-weight baselines. 

However, when we replicated the tests with expanded datasets including more dates and using different assets, we noticed that performance was significantly worse. This was likely due to the model overfitting of the time period and assets presented in the paper. The model performed particularly poorly in the 2020-2024 timeframe, and significantly worse than the benchmark. Values are presented in the table below. 

#### Enhancements
We aimed to enhance the Zhang et al. approach by exploring alternative designs and data, addressing potential limitations. This included using a richer historical dataset, integrating state variables for improved regime learning, and exploring LSTM ensemble with state variables and transformer architectures like encoder-only and encoder-decoder models. Building on promising results, we chose the full transformer architecture (Transformer), a sequence-to-sequence model with configurable dimensions and attention mechanisms.

<div style="text-align: center;">
  <img src="/assets/images/port_opt/transformer_architecture.png" alt="Transformer Architecture" width="500"/>
  <figcaption>Transformer Architecture</figcaption>
</div>

In these experiments, we implemented the Sharpe loss function like the Zhang et al. experiments. To enhance model robustness, we added L2 Regularization to the Adam optimizer and included a dropout layer for normalization and to counter overfitting. Hyperparameter tuning used a 10% validation set from each training period, with testing on a search grid.

### Results
Our final experiment aimed to evaluate whether pre-training, expanded features, and the transformer architecture could outperform the Zhang et al. LSTM model. The Transformer achieved a Sharpe ratio of 2.7, surpassing others in risk metrics, and returns. It consistently outperformed LSTM, MVO, and Balanced portfolios, demonstrating robustness and effectiveness in portfolio optimization. Specifically, the Transformer model produced higher rolling Sharpe ratios on 0.72, 0.90, and 0.92 of the days over the entire test period versus LSTM, MVO, and Balanced, respectively, and 0.68, 0.69, and 0.74, respectively, of the days during the zoom-in period post 05/01/2020. 

<div style="text-align: center;">
  <img src="/assets/images/port_opt/table.png" alt="Table" width="750"/>
  <figcaption>Model Metrics</figcaption>
</div>

<figure class="half">
    <a href="/assets/images/port_opt/transformer_benchmark.png"><img src="/assets/images/port_opt/transformer_benchmark.png"></a>
    <a href="/assets/images/port_opt/transformer_oxford.png"><img src="/assets/images/port_opt/transformer_oxford.png"></a>
    <figcaption>1 Year Rolling Sharpe of Transformer vs Benchmarks </figcaption>
</figure>