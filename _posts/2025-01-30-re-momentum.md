---
title: "A Metro-Level Real Estate Momentum Trading Strategy"
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

[View Interactive Dashboard](https://real-estate-momentum.streamlit.app/)

### Introduction

Momentum trading strategies have been extensively studied and implemented across various asset classes, such as equities, fixed income, and commodities. The premise is that assets that have performed well in the recent past tend to continue performing well in the near future, and vice versa.

Single family real estate markets exhibit distinct characteristics from these markets, particularly in their price discovery mechanisms and market efficiency. This phenomenon can be attributed to several structural factors: high transaction costs, information asymmetry, and behavioral biases in residential property decisions. This results in real estate markets being characterized by more lagged price movements. 

Looking at housing markets across major U.S. cities, when home prices start rising in an metro area, they tend to continue rising for a while. This makes sense, since the key factors that drive home values - like job growth, demographics and environment - gradually change over time. This slow-moving nature of real estate fundamentals hints that past price trends in a city might help predict where prices are headed next.

We explore whether past price trends in different metro housing markets can reliably predict future returns. By analyzing historical data across major U.S. cities, we aim to test if a systematic momentum strategy could work in residential real estate.

### Methodology

#### Data Sources and Processing

To construct the metro-level momentum factor, we utilize housing price data from Zillow's Home Value Index (ZHVI), which provides monthly median home values across major U.S. metropolitan statistical areas (MSAs).

#### Defining the Metro-Level Momentum Factor

**Simple Momentum**:
For simplicity, we define a straightforward momentum indicator to test this trading strategy. The indicator ranks metro areas based on their trailing 12-month price appreciation, allowing us to construct portfolios of high momentum markets.

For each metro area i at time t, we calculate the trailing X-period price appreciation:

$$
Momentum_{i,t} = \frac{Price_{i,t}}{Price_{i,t-X}} - 1
$$

This simple measure allows us to rank metros based on their recent price performance.

**LSTM-Based Momentum**: 
Rather than imposing a specific lookback window, we feed the LSTM model with the full price history for each metro. This allows the model to learn complex temporal patterns and potentially identify more nuanced momentum signals.

### Statistical Framework and Model

Our analysis starts with testing a straightforward momentum strategy. We rank metropolitan areas based on their trailing price appreciation over X months and invest in the top 10% of metros showing the strongest momentum. At each rebalancing period, we recalculate these rankings and adjust our portfolio accordingly, holding positions for Y months. This simple approach lets us test whether past winners tend to remain winners in residential real estate markets.

To evaluate performance, we compare this momentum portfolio against a benchmark of the average US home price. 

### Analysis to follow