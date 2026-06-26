# Background: the SentioEdge / SPECTRA review

This file records the analysis that motivated Project SENTINEL, so the design
rationale isn't lost.

## Independent-reputation check (as of mid-2026)

- **Reddit:** no substantive discussions from traders who have actually used
  SentioEdge / SPECTRA — notable, since established TradingView products
  (LuxAlgo, AlgoAlpha, Market Cipher, etc.) accumulate many community threads.
- **Trustpilot:** ~6 reviews, ~3.9/5; nearly every positive review from a
  single-review account; one reviewer questioned authenticity on that basis.
- **Not found:** independent YouTube tests, long-term journals, MyFxBook-style
  verification, ForexFactory threads, independent comparisons. Most results
  point back to the vendor's own marketing.

**Classification:** high marketing presence, very limited independent feedback,
no verifiable long-term performance, community reputation too new/small to judge.

**Before buying, ask the vendor for:** live trades (not screenshots), a public
TradingView strategy with transparent settings, 6–12 months of forward-tested
results, performance per market (Gold/FX/Indices/Crypto), and win rate / avg R /
max drawdown / trade count.

## Reverse-engineering SPECTRA from its marketing

SPECTRA's disclosed pipeline:

```
Price → Kalman Filter → Supertrend → Smart Trail → RSI filter → Volume filter → Signal
```

Assessment: **not** ML / neural nets / LLMs / predictive AI — "AI-powered" reads as
marketing. It's deterministic, classical quantitative TA, well integrated.

| Component        | What it is                                   | Originality |
|------------------|----------------------------------------------|-------------|
| Kalman filter    | Low-lag price smoothing before Supertrend    | 7/10 |
| Supertrend       | Trend detector (ATR 10–14, mult 2.5–4)       | 2/10 |
| Smart Trail      | ATR trail / Chandelier-style confirmation    | 3/10 |
| RSI filter       | Momentum gate (~50/55 thresholds)            | 1/10 |
| Volume filter    | Normal vs Strong via volume > SMA(vol,20)    | 2/10 |
| Trend cloud      | Fast vs slow EMA fill (e.g. 20/50 or 34/89)  | — |
| Reversal zones   | VWMA + deviation bands (exhaustion, not predictive) | — |
| Trade management | Swing ± ATR stop, TP1 = 1R, TP2 = 2R         | good |
| Dashboard        | Trend/strength/momentum/vol from above       | 5/10 |
| Backtest table   | Mini strategy tester (win%, PF, etc.)        | 8/10 |
| **Integration**  | The actual value: polished, coherent workflow| 8/10 |

**Takeaway:** the ingredients are common; the value is integration + presentation.
For a market-structure / liquidity-sweep / FVG / HTF-confluence trader, taking
SPECTRA signals mechanically is a poor fit — better to build a purpose-made tool.
Hence SENTINEL.
