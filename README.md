# Trader-Performance-vs-Market-Sentiment-Analysis

# Market Sentiment vs Trader Behavior & Performance - Hyperliquid

Analysis of how Bitcoin Fear/Greed sentiment relates to trader behavior and
performance on Hyperliquid, using trade-level history for 32 accounts and a
daily Fear/Greed index.

## What's in this repo

```
notebook.ipynb        Full analysis, executed top-to-bottom (Parts A, B, C + Bgonus)
WRITEUP.md             1-page methodology / insights / strategy summary
README.md              this file
data/
  historical_data.csv       raw Hyperliquid trade history (input)
  fear_greed_index.csv      raw daily Fear/Greed classification (input)
charts/                 all charts, exported as PNG (also embedded in the notebook)
```

## Setup

Requires Python 3.10+.

```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy jupyter
```

## How to run

Option A — open and run the notebook:

```bash
jupyter notebook notebook.ipynb
```

Option B — run headless and regenerate all outputs in place:

```bash
jupyter nbconvert --to notebook --execute --inplace notebook.ipynb
```

The notebook reads both CSVs from `./data/`, so run it from the repo root (or
adjust `RAW_DIR` in the first code cell).

## Method summary

1. **Load & document** both raw files (shape, dtypes, missing values, duplicates).
2. **Align by date**: timestamps parsed, trades tagged with the same-day Fear/Greed
   classification (479/480 trading days matched; 1 unmatched day dropped from
   sentiment comparisons).
3. **Build an account-day metrics table** — the core analysis unit — with: trade
   count, total/average trade size (USD), realized PnL net of fees, win rate
   (computed over *closing* trades only, since opens carry `Closed PnL == 0`),
   and long/short volume split.
4. **Compare Fear vs Greed** on performance (PnL, win rate, a tail-loss drawdown
   proxy, Mann-Whitney significance test) and behavior (trade frequency, size,
   long/short bias).
5. **Segment traders** three ways: trade-size (high/low), frequency
   (frequent/infrequent), and consistency (consistent winners vs inconsistent),
   and re-cut performance by regime within each segment.
6. **Bonus**: KMeans clustering into 3 trader archetypes, and a simple
   logistic-regression / random-forest baseline predicting next-day
   profitability from sentiment + the account's own same-day behavior.

## Known data-quality note

The assignment brief mentions a `leverage` field, but the actual
`historical_data.csv` does not contain one. Rather than invent a number, this
analysis uses **trade notional (`Size USD`)** as an explicit exposure proxy
wherever leverage-style analysis is requested, and calls this out inline.

## Key findings (see `WRITEUP.md` for the full summary)

- Median daily PnL per account is higher on Greed days, but Fear days show
  fatter tails in both directions (Mann-Whitney p ≈ 0.016).
- Traders are **more** active and trade **larger** size on Fear days, not less.
- Long positioning rises as fear intensifies (56% of volume at Extreme Greed →
  76.5% at Extreme Fear) — contrarian/dip-buying behavior, not panic-selling.
- Segment (size / frequency / consistency) explains more PnL variation than
  sentiment regime alone — confirmed by the modest AUC (~0.62–0.65) of the
  bonus predictive model.
