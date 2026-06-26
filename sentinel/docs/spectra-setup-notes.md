# SPECTRA setup-video notes (reference)

Captured from the SentioEdge/SPECTRA YouTube setup walkthrough. Used as design
input for SENTINEL — see how each maps to our build at the bottom.

## What it is
Signal-processing indicator for TradingView (invite-only). Reads market
conditions in real time, prints buy/sell signals on any instrument/timeframe,
with automatic TP/SL levels and a built-in backtest table.

**Most important property: it does not repaint.** Every signal is locked at
candle close — historical = what you'd have seen live.

**Golden rule:** must be tuned individually for *every* instrument AND timeframe.
A BTC 1h config won't work on Gold 4h. Budget 5–10 min per new setup.

## Mechanics
- Price + momentum + volume filters as confluence before a signal fires.
- Normal signals plus Strong Buy / Strong Sell (higher conviction).
- Stop loss from recent S/R lookback; take-profit targets drawn as boxes/lines.
- Tune params while watching the backtest table update live, then trust it.

## Settings sections
- **Signal Engine** — core tuning: Signal Sensitivity, Market Noise Period,
  Signal Smoothing, Smart Trail Sensitivity, Momentum Period, Momentum/Volume
  filter toggles (keep both ON).
- **TP/SL Management** — Stop Loss Lookback, Stop Loss Buffer, Show TP/SL,
  TP1/TP2. Preferred: both TPs, split position in half (½ at TP1, ½ runs to TP2).
- **Tables & Dashboards** — market-condition + backtest displays.
- **Overlays** (visual only, no signals) — Smart Trail, Trend Cloud, Reversal Zones.
- **Style/Visibility** — cosmetic only.

## Backtest table
- Overall trades + combined results; W/L; WR%; P&L (ticks).
- **Profit Factor = the key number** (profits/losses). >1.0 profitable; aim ≥1.2.
- Buy vs Sell breakdown (reveals directional bias on an instrument).
- Outcomes: TP1+TP2 (best), TP1+SL (~breakeven), SL (full loss), Exit/BE.

## Extra tools
- **Arrow-Way Exit** — if in profit and a new opposite signal fires, marks a
  yellow exit arrow: lock profit and flip.
- **Spectra Dashboard** — real-time bias + active-signal health check.
- **Multi-Timeframe Panel** — trend strength across timeframes.
- **Alerts** — all / buy-only / sell-only / strong-only.

## Workflow
Add → enable backtest table → keep momentum+volume ON → tune Signal Engine
watching Profit Factor (≥1.2) and buy/sell split → set TP1/TP2 (half/half) +
SL lookback/buffer → add overlays/dashboards → set alerts → re-tune per
instrument/timeframe.

---

## How this maps to SENTINEL

| SPECTRA feature              | SENTINEL status |
|------------------------------|-----------------|
| No-repaint (lock on close)   | **v1.1** — `confirmClose` toggle |
| Momentum (RSI) filter        | **v1.1** — confluence point |
| Volume filter                | **v1.1** — confluence point |
| Normal vs Strong signals     | **v1.1** — `strongConf` threshold |
| Alert modes (all/buy/sell/strong) | **v1.1** |
| Multi-timeframe bias panel   | v1 — HTF bias dashboard |
| TP/SL from S/R lookback + buffer | **v2** — trade manager |
| TP1/TP2 split                | **v2** |
| Backtest table (PF, WR, split, outcomes) | **v2** (needs TP/SL) |
| Arrow-Way Exit               | **v2** (needs position state) |
| Smart Trail / Trend Cloud / Reversal Zones | v3 overlays |
| Per-instrument tuning ethos  | documented in README |
