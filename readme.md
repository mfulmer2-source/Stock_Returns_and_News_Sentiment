# Stock Returns and News Sentiment Analysis


## Overview

This project analyzes the relationship between weekly stock returns and
news sentiment for ten major technology companies using data from Alpha
Vantage. The goal is to determine whether news sentiment provides
meaningful information about same-week or next-week stock performance.

Weekly adjusted closing prices were collected for 10 large-cap tech
firms, and thousands of news articles were retrieved from Alpha
Vantage’s NEWS_SENTIMENT endpoint for the 2020–2025 period. Sentiment
scores were aggregated to weekly averages and merged with weekly stock
returns to evaluate potential relationships and predictive patterns.

The sections below outline the data, methods, selected code used in the
analysis, and key results.

------------------------------------------------------------------------

## Data Sources

### Stock Prices

- **Provider:** Alpha Vantage (TIME_SERIES_WEEKLY_ADJUSTED)  
- **Tickers:** NVDA, AAPL, MSFT, GOOGL, AVGO, META, TSM, ORCL, PLTR,
  ASML  
- **Frequency:** Weekly  
- **Variables:** Adjusted close, open, high, low, volume, dividend

### News Sentiment

- **Provider:** Alpha Vantage (NEWS_SENTIMENT)
- **Filters:** technology, earnings (2020–present)
- **Returned variables:**
  - overall sentiment score  
  - sentiment label  
  - ticker-specific sentiment score  
  - title, source, timestamp

### Data Limitations

- Alpha Vantage caps sentiment queries at **1,000 articles per ticker**,
  leaving some stocks with fewer usable weeks.
- Many articles mention a ticker only once, introducing noise.
- Final analysis focuses on **AAPL, META, ORCL, PLTR** because each has
  50+ weeks of overlapping sentiment and return data.

## Repository Structure

This repository contains the full workflow, processed data, and written
report for the final project.

- `Final_Project.qmd` — main analysis file containing all code for data
  collection, cleaning, merging, and visualization  
- `README.md` — rendered written report in GitHub Markdown  
- `all_prices.pkl` — saved and cleaned weekly stock price data  
- `news.pk2` — processed and cleaned news sentiment data

*Note: To re-run the data download steps inside `Final_Project.qmd`, you
must insert your own Alpha Vantage API key (e.g.,
`av_k = "YOUR_API_KEY_HERE"`). The saved datasets provided
(`all_prices.pkl` and `news.pk2`)allow full reproduction of the analysis
without re-downloading data.*

------------------------------------------------------------------------

## Methods

Weekly stock returns were calculated using percentage changes in
adjusted closing prices. Sentiment was converted into weekly averages
per company. Datasets were merged by ticker and week. Same-week and
next-week correlations were computed to test predictive relationships.

Representative code examples are shown below.

------------------------------------------------------------------------

## Load and Prepare Price Data

The following chunk loads the weekly price data and displays the
structure.

``` python
import pandas as pd
import pickle

df = pd.read_pickle("all_prices.pkl")
df.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | date | open | high | low | close | adj_close | volume | dividend | symbol |
|----|----|----|----|----|----|----|----|----|----|
| 1361 | 2025-12-01 | 278.010 | 283.420 | 276.140 | 283.10 | 283.1000 | 45452514.0 | 0.00 | AAPL |
| 1362 | 2025-11-28 | 270.900 | 280.380 | 270.900 | 278.85 | 278.8500 | 166067059\.0 | 0.00 | AAPL |
| 1363 | 2025-11-21 | 268.815 | 275.430 | 265.320 | 271.49 | 271.4900 | 235974430\.0 | 0.00 | AAPL |
| 1364 | 2025-11-14 | 268.960 | 276.699 | 267.455 | 272.41 | 272.4100 | 232952837\.0 | 0.26 | AAPL |
| 1365 | 2025-11-07 | 270.420 | 273.400 | 266.250 | 268.47 | 268.2112 | 241487127\.0 | 0.00 | AAPL |

</div>

Next, prices are limited to the 2020–present period and weekly returns
are computed.

``` python
df_2020 = df[df['date'] >= '2020-01-01'].copy()
df_2020['returns'] = df_2020.groupby('symbol')['adj_close'].pct_change()
df_2020.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | date | open | high | low | close | adj_close | volume | dividend | symbol | returns |
|----|----|----|----|----|----|----|----|----|----|----|
| 1361 | 2025-12-01 | 278.010 | 283.420 | 276.140 | 283.10 | 283.1000 | 45452514.0 | 0.00 | AAPL | NaN |
| 1362 | 2025-11-28 | 270.900 | 280.380 | 270.900 | 278.85 | 278.8500 | 166067059\.0 | 0.00 | AAPL | -0.015012 |
| 1363 | 2025-11-21 | 268.815 | 275.430 | 265.320 | 271.49 | 271.4900 | 235974430\.0 | 0.00 | AAPL | -0.026394 |
| 1364 | 2025-11-14 | 268.960 | 276.699 | 267.455 | 272.41 | 272.4100 | 232952837\.0 | 0.26 | AAPL | 0.003389 |
| 1365 | 2025-11-07 | 270.420 | 273.400 | 266.250 | 268.47 | 268.2112 | 241487127\.0 | 0.00 | AAPL | -0.015414 |

</div>

## Prepare Sentiment Data

News sentiment is loaded and aggregated into weekly averages.

``` python
news = pd.read_pickle("news.pk2")
news.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | symbol | date | overall_sentiment_score | overall_label | ticker_sentiment | ticker_sent_label | title | source | week |
|----|----|----|----|----|----|----|----|----|----|
| 0 | NVDA | 2025-12-02 19:00:00 | 0.453751 | Bullish | 0.456727 | Bullish | KLA Surges: AI Chip Demand Fuels Stock Perform... | Markets Financial Content | 2025-11-28 |
| 1 | NVDA | 2025-12-02 18:20:00 | 0.287599 | Somewhat-Bullish | 0.319854 | Somewhat-Bullish | KLA, IBD Stock Of The Day, Flashes Buy Signal ... | Investor's Business Daily | 2025-11-28 |
| 2 | NVDA | 2025-12-02 14:05:00 | 0.140530 | Neutral | 0.001000 | Neutral | Transcript : Cisco Systems, Inc. Presents at U... | MarketScreener | 2025-11-28 |
| 3 | NVDA | 2025-12-02 12:36:21 | 0.413756 | Bullish | 0.494082 | Bullish | How Is KLA Corporation’s Stock Performance Com... | MSN | 2025-11-28 |
| 4 | NVDA | 2025-12-02 12:33:00 | 0.399579 | Bullish | 0.341557 | Somewhat-Bullish | The Top 5 Analyst Questions From Analog Device... | Finviz | 2025-11-28 |

</div>

``` python
weekly_sentiment = (
news.groupby(['symbol', 'week'])['ticker_sentiment']
.mean()
.reset_index()
)

weekly_sentiment.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|     | symbol | week       | ticker_sentiment |
|-----|--------|------------|------------------|
| 0   | AAPL   | 2024-08-23 | 0.021634         |
| 1   | AAPL   | 2024-08-30 | 0.108780         |
| 2   | AAPL   | 2024-09-06 | 0.001000         |
| 3   | AAPL   | 2024-09-20 | 0.220137         |
| 4   | AAPL   | 2024-09-27 | 0.066498         |

</div>

## Create Weekly Period in Price Data

Stock price dates are already weekly, so the week variable is assigned
directly from the date.

``` python
df_2020['week'] = df_2020['date']
```

## Merge Sentiment and Returns

Below is the merge that links weekly returns with weekly sentiment.

``` python
week_and_return = df_2020[['symbol', 'week', 'returns']]
merged = weekly_sentiment.merge(week_and_return, on=['symbol','week'], how='left')
merged.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|     | symbol | week       | ticker_sentiment | returns   |
|-----|--------|------------|------------------|-----------|
| 0   | AAPL   | 2024-08-23 | 0.021634         | -0.009432 |
| 1   | AAPL   | 2024-08-30 | 0.108780         | 0.037044  |
| 2   | AAPL   | 2024-09-06 | 0.001000         | -0.007551 |
| 3   | AAPL   | 2024-09-20 | 0.220137         | 0.001800  |
| 4   | AAPL   | 2024-09-27 | 0.066498         | 0.004365  |

</div>

Restrict the sample to the four main companies used in the analysis.

``` python
symbol_remove = ['ASML', 'AVGO', 'GOOGL', 'MSFT', 'NVDA', 'TSM']

four_companies = merged[~merged['symbol'].isin(symbol_remove)]
four_companies
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|     | symbol | week       | ticker_sentiment | returns   |
|-----|--------|------------|------------------|-----------|
| 0   | AAPL   | 2024-08-23 | 0.021634         | -0.009432 |
| 1   | AAPL   | 2024-08-30 | 0.108780         | 0.037044  |
| 2   | AAPL   | 2024-09-06 | 0.001000         | -0.007551 |
| 3   | AAPL   | 2024-09-20 | 0.220137         | 0.001800  |
| 4   | AAPL   | 2024-09-27 | 0.066498         | 0.004365  |
| ... | ...    | ...        | ...              | ...       |
| 336 | PLTR   | 2025-10-31 | 0.207593         | 0.126679  |
| 337 | PLTR   | 2025-11-07 | 0.188086         | 0.022527  |
| 338 | PLTR   | 2025-11-14 | 0.163168         | 0.123733  |
| 339 | PLTR   | 2025-11-21 | 0.136246         | -0.080736 |
| 340 | PLTR   | 2025-11-28 | 0.171514         | 0.005732  |

<p>295 rows × 4 columns</p>
</div>

## Visualization Examples

The following plot shows weekly adjusted prices for all companies from
2020–present.

``` python
import plotly.express as px

fig_prices = px.line(
df_2020,
x='date',
y='adj_close',
color='symbol',
title='Weekly Adjusted Closing Prices (2020–Present)'
)

fig_prices
```

Next is the same-week sentiment vs. returns scatterplot with trendlines.

``` python
fig4 = px.scatter(
four_companies,
x='ticker_sentiment',
y='returns',
color='symbol',
facet_col='symbol',
facet_col_wrap=2,
trendline='ols',
opacity=0.5,
title='Weekly Returns vs News Sentiment'
)

fig4
```

## Results

### Weekly Stock Price Trends

Major technology stocks displayed strong growth and volatility from
2020–2025, particularly NVDA and AAPL.

### Weekly Returns

Weekly returns were noisy and showed company-specific volatility
patterns.

### Article Volume and Sentiment

- META and AAPL had the most news coverage.
- Sentiment distributions were dominated by Neutral and Somewhat-Bullish
  articles, with Bullish appearing less frequently.

### Same-Week Sentiment vs Returns

Correlations were consistently weak and negative across all four
companies:

| Ticker | corr(sentiment, returns) |
|--------|--------------------------|
| AAPL   | –0.067                   |
| META   | –0.196                   |
| ORCL   | –0.189                   |
| PLTR   | –0.278                   |

### Lagged Sentiment vs Next-Week Returns

Lagged correlations remained small:

| Ticker | corr(sentiment, next-week returns) |
|--------|------------------------------------|
| AAPL   | –0.13                              |
| META   | +0.09                              |
| ORCL   | +0.06                              |
| PLTR   | –0.20                              |

------------------------------------------------------------------------

## Discussion

Short-term movements in large-cap technology stocks show **minimal
association** with news sentiment scores from Alpha Vantage. Several
factors likely contribute:

- Articles often mention multiple companies, reducing relevance.  
- Weekly aggregation may smooth out meaningful signals.  
- Tech stocks respond strongly to earnings, macroeconomic news, and
  industry events.

### Future Improvements

- Filter only articles where the ticker appears in the **title**.  
- Weight sentiment by article **relevance score**.  
- Use **daily** sentiment instead of weekly averages.  
- Expand beyond the **1,000-article limit** imposed by Alpha Vantage.

------------------------------------------------------------------------

## Conclusion

This project implemented a complete pipeline for gathering, merging, and
analyzing stock returns and news sentiment. Despite intuitive
expectations, sentiment showed **little predictive power** for same-week
or next-week returns. The results highlight the challenges of using
aggregated sentiment for short-term financial forecasting and point
toward opportunities for improved filtering, more granular data, and
enhanced modeling strategies.
