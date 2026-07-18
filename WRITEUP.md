# Write-up: Sentiment vs Trader Behavior on Hyperliquid

## Methodology

Two files were merged on calendar date: Hyperliquid trade history (211,224 rows,
32 accounts, 480 trading days, no missing values or duplicates) and a daily
Fear/Greed index (2,644 days, 5-way classification, no missing values or
duplicates). 479 of 480 trading days matched a sentiment label; the one unmatched
day was dropped from sentiment comparisons. The 5-way classification was
collapsed to a binary **Fear** (Fear + Extreme Fear) / **Greed** (Greed +
Extreme Greed) regime for the core comparisons, keeping the full 5-way scale for
finer-grained charts.

The core analysis unit is an **account-day**: trade count, total/average trade
size (USD notional), realized PnL net of fees, win rate, and long/short volume
split, all aggregated per account per day and joined to that day's sentiment.
Win rate is computed only over *closing* trades (Close Long/Short, Sell,
liquidation/settlement events) since opening trades carry `Closed PnL == 0` by
construction on Hyperliquid — including opens in a naive win-rate would silently
dilute it. **The source data has no `leverage` column** despite the brief
mentioning one; `Size USD` is used as an explicit, labelled exposure proxy
wherever a leverage-style view is called for.

Three trader segments were built from account-level aggregates: size
(high/low, split at the median average trade size), frequency
(frequent/infrequent, split at median active trading days), and consistency
(consistent winners = ≥55% profitable days and positive total PnL, among
accounts with ≥5 active days, to avoid labelling accounts with too few
observations). A Mann-Whitney U test checked whether Fear/Greed PnL
distributions differ significantly. Two bonus models were added: KMeans
clustering (k=3, chosen via inertia elbow) into behavioral archetypes, and a
logistic regression / random forest predicting an account's next active-day
profitability from that same day's sentiment + its own behavior (no lookahead
leakage — only same-day, already-realized information is used as input).

## Insights

1. **Fear does not suppress trading — it concentrates it.** Accounts traded
   ~37% more often (105 vs 77 trades/account/day) and at ~43% larger average
   size on Fear vs Greed days. Median PnL per account-day is higher on Greed
   days, but Fear days show fatter tails (Mann-Whitney p ≈ 0.016) and a
   slightly worse tail-loss proxy (avg of worst 5% of account-days: −21.4k vs
   −19.0k).
2. **Long positioning rises as fear intensifies, not the reverse.** Long share
   of trading volume climbs from 56.1% at Extreme Greed to 76.5% at Extreme
   Fear — this dataset shows contrarian, dip-buying behavior rather than panic
   liquidation as sentiment worsens.
3. **Segment membership swings PnL by 2–4x within a single sentiment regime** —
   far more than sentiment itself. High-size accounts earn ~9.2k mean net PnL
   on Fear days vs 3.2k on Greed; infrequent traders out-earn frequent traders
   in both regimes; and "consistent winners" hold up better specifically on
   Fear days (6.0k vs 2.9k for Greed), while inconsistent traders' apparent edge
   is concentrated in Greed conditions. This is corroborated by the bonus
   model: sentiment features rank low in feature importance, and AUC (~0.62–
   0.65) is only modestly above random — an account's own recent PnL/win rate
   predicts its near-term future better than the sentiment label does.

## Strategy recommendations

1. **Don't apply a blanket "de-risk in Fear" rule — it penalizes the segment
   that performs best there.** High-notional and infrequent/selective traders
   show their strongest relative edge on Fear days in this data. A rule like
   "reduce size across the board when Fear/Extreme Fear prints" would cut
   exposure exactly where the historical edge is largest for those segments.
   Rule of thumb: **gate any Fear-day size reduction by segment** — apply it to
   high-frequency/low-size accounts (whose edge doesn't show a Fear premium),
   not uniformly.
2. **Treat "Extreme Fear + already-elevated long share" as a crowding flag, not
   a clean contrarian entry.** Because long positioning is already highest
   exactly when fear is most extreme, a pure "buy when everyone is fearful"
   heuristic risks entering alongside a crowd that has pre-empted the dip-buy
   thesis. Rule of thumb: **size contrarian entries down (not up) when Extreme
   Fear coincides with long share already above its regime average**, and save
   full-size contrarian entries for Fear readings where long share is still
   closer to neutral.

## Limitations

32 unique accounts and ~1 year of overlapping history is a small, non-random
sample (likely selected/whale-heavy Hyperliquid users) — segment and cluster
splits are directional signals, not statistically robust claims, and shouldn't
be extrapolated to the broader trader population or to other assets/venues
without re-validation.
