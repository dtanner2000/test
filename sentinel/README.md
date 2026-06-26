# Project SENTINEL

**Smart Execution for Institutional Liquidity** — a modular SMC / institutional-style
trading framework for TradingView (Pine Script v6).

This project grew out of a review of the commercial indicator **SentioEdge / SPECTRA**.
The conclusion of that review (summarised in [`docs/background.md`](docs/background.md))
was that SPECTRA is a polished combination of *classical* techniques — Kalman smoothing,
Supertrend, RSI/volume filters, ATR trailing stops — rather than anything genuinely
"AI-powered". Rather than copy a generic trend-follower, SENTINEL is built for an
institutional / Smart-Money-Concepts workflow: higher-timeframe bias → institutional
points of interest → wait for price → confirm the reaction → score the setup.

> ⚠️ **Not financial advice.** This is a decision-support tool. Signals are confluences,
> not guarantees. Forward-test on a demo account before risking capital.

---

## Status

| Version | Theme        | State          |
|---------|--------------|----------------|
| **v1**  | Foundation   | ✅ Shipped |
| **v2**  | Execution    | ✅ Shipped |
| **v3**  | Professional | ✅ Shipped |
| **v4**  | Elite        | ✅ **Shipped** (`SENTINEL_v1.pine` — single evolving script) |

### What's in the box

**Foundation (v1):**
1. **Higher-Timeframe Bias** — Daily / 4H / 2H, each scored bull/bear/flat vs an EMA, summed to a net bias (−3…+3).
2. **Fair Value Gaps** — automatic bullish/bearish FVG detection, drawn as boxes, with a full mitigation lifecycle (extend → consume → delete/grey-out).
3. **Liquidity engine** — sweep / stop-hunt detection (wick takes prior swing, closes back inside).
4. **Market structure** — BOS / CHOCH derived from swing pivots, with a running trend state.
5. **Session intelligence** — Asian / London / New York kill-zone shading + PDH/PDL and PWH/PWL liquidity levels.
6. **Confirmation filters** — RSI momentum + volume participation, added as confluences (keep both ON).
7. **Confluence scoring** — dynamic `X/maxConf` long/short score → **Buy/Sell + STRONG** signals, **no-repaint** commit-on-close, alert routing (all/buy/sell/strong), template `alertcondition()`s, and a compact on-chart dashboard.

**Execution (v2):**
8. **Trade manager** — on each confirmed signal: entry, stop (recent S/R lookback ± ATR buffer), TP1/TP2 (R multiples), drawn as red SL / green TP zones with a dashed TP1 line. Half-and-half management (½ at TP1, ½ runs to TP2).
9. **Arrow-Way / Smart Exit** — closes the active trade early when an opposite signal fires (yellow `exit` marker).
10. **Backtest table** — W/L, **win-rate**, P/L, **Profit Factor**, broken down by OVERALL / All BUY / BUY / Strong BUY / All SELL / SELL / Strong SELL, plus an OUTCOMES section (TP1+TP2 / TP1+SL / SL / Exit-BE with count and P/L). Mirrors SPECTRA's layout so you can tune watching PF (aim ≥1.2) and the buy/sell split.

> ⚠️ **The backtest is an indicator-side *simulation*** (½ at TP1, ½ to TP2; stop checked pessimistically before targets within a bar). It's an estimate for tuning — not a broker-accurate fill model. Treat it as directional, not gospel.

**Professional (v3):**
11. **Order-block engine** — demand/supply OBs auto-detected as the last opposite-colour candle before a structure break, drawn as zones with a mitigation lifecycle; when price closes through an OB it flips to a **breaker** (orange) or is deleted.
12. **Confidence grade** — the active confluence score is mapped to an **A+ / A / B / C / D** grade, shown in the dashboard.
13. **Visual overlays** (default OFF, enable when comfortable) — **Smart Trail** (ATR trailing stop), **Trend Cloud** (fast/slow EMA fill), **Reversal Zones** (VWMA ± deviation bands).

**Elite (v4):**
14. **Kalman trend filter** — a 1-D Kalman filter on price; rising/falling adds an optional confluence point (so `maxConf` becomes 8 with it on) and can be plotted.
15. **Market-regime classifier** (deterministic, no ML) — labels the tape **Trend / Range / Breakout / Reversal** from ADX, ATR expansion, structure shifts and reversal-zone extremes.
16. **POI ranking** — shows the nearest demand/supply order block aligned with the current bias, with its price, in the dashboard.
17. **Multi-symbol scanner** (default OFF) — bias + RSI for up to 5 symbols on a chosen timeframe, in its own table.

> **Golden rule (from the SPECTRA setup video, applies here too):** tune per
> instrument *and* per timeframe — a BTC 1h config won't suit Gold 4h. Budget a
> few minutes to dial in sensitivity (pivot length, confluence thresholds,
> RSI/volume) for each new chart.

---

## Changelog

- **v4.8.0** — aligned the **indicator** with the validated strategy: added the **HTF-bias-alignment** filter (default ON) + a kill-zone-only toggle to the indicator's signal logic, so its on-chart backtest reflects the same trend-filtered trades the strategy validated (no more counter-trend signals dragging it down). Also made the normal Buy/Sell markers larger (small triangles with B/S text) so they're visible over the overlays.
- **v4.7.0** — re-baked both scripts' default inputs to the **12-year validated USTEC 20m config** (PF 1.149, 2,804 trades): biasLen 101, HTF 1M/2W/1W, pivot 5, RSI 13, volume filter off, conf 4/4/5, SL 15 / ATR 14 / buffer 0.5, TP1 1.6 / TP2 3.2; strategy adds risk 1.8% + HTF-bias-alignment filter ON. "Reset settings to defaults" now loads it.
- **v4.6.0** — strategy robustness toolkit: **Smart Trail** exit mode (½ TP1, rest trails an ATR stop), entry **filters** (require HTF bias alignment / kill-zones only / min-ADX), and **risk caps** (daily-loss limit %, max-drawdown halt %). Added HTF-per-timeframe guidance for testing higher timeframes.
- **v4.5.0** — added **`SENTINEL_strategy.pine`** for forward testing: same signal engine, executed via `strategy.*` so the native Strategy Tester gives broker-accurate stats and can drive paper/live via alerts. Risk 1.5%/trade on $50k, ½TP1/½TP2 + shared stop, Arrow-Way reversal, orders on bar close. Recorded the locked USTEC 5m config (≈PF 1.41) in *Tuned configs*.
- **v4.4.0** — **removed the code-baked preset system** (reverting v4.2). A Pine preset works by *overriding* inputs in code, which makes those input boxes dead/non-responsive while a preset is active — confusing and easy to mistake for a bug. All inputs are now always live again. Save per-instrument configs with TradingView's **native** "Save as Default" / chart-layout-per-symbol instead (see *Saving tuned configs* below). The USTEC 5m values are preserved in this README.
- **v4.3.0** — cleaner signal markers (SPECTRA-style): normal signals are now small triangles, strong signals are compact `BUY+` / `SELL+` pills (replacing the bulky "STRONG" text).
- **v4.2.0** — instrument presets:
  - **Preset dropdown** (`★ Preset`) — `Manual` or `USTEC 5m`. Selecting a preset overrides the key engine parameters with a tuned set; the dashboard badge shows the active preset.
  - **USTEC 5m** preset values (≈PF 1.36 in testing): biasLen 25, HTF3 30m, RSI 20, volume avg 19, min-conf long 5 / short 4, TP2 3.2R, reversal-zone length 28 (all other params at default).
  - Implemented via a resolver layer (inputs renamed `*_i`, resolved into the working names), so presets compose cleanly and more pairs can be added with one ternary branch each.
- **v4.1.0** — tuning controls (from analysing live backtests, e.g. USTEC strong-shorts dragging PF):
  - **Trade-direction filter** — Both / Long only / Short only, to suppress one side on instruments with a structural drift (e.g. indices).
  - **Separate short-side min confluence** (`minConfS`) — raise the bar for shorts independently of longs. Defaults equal to the long threshold, so behaviour is unchanged until adjusted.
- **v4.0.0** — Elite release (confirmed v3.0 runs clean on TradingView):
  - **Kalman trend filter** — 1-D Kalman on price, added as an optional confluence (max becomes 8) and plottable.
  - **Market-regime classifier** — Trend / Range / Breakout / Reversal from ADX + ATR expansion + structure + reversal zones.
  - **POI ranking** — nearest bias-aligned order block shown in the dashboard.
  - **Multi-symbol scanner** (default OFF) — bias + RSI for up to 5 symbols.
  - Completes the v1–v4 roadmap.
- **v3.0.0** — Professional release (confirmed v2.0 runs clean on TradingView):
  - **Order-block engine** — demand/supply zones from the last opposite candle before a structure break, with mitigation + breaker lifecycle.
  - **Confidence grade** — A+/A/B/C/D from the confluence score, shown in the dashboard.
  - **Overlays** (default OFF) — Smart Trail (ATR trailing stop), Trend Cloud (EMA fill), Reversal Zones (VWMA ± deviation bands).
- **v2.0.0** — Execution release (confirmed v1.1 runs clean on TradingView):
  - **Trade manager** — entry / stop (S/R lookback ± ATR buffer) / TP1 / TP2 (R multiples), red SL & green TP zones + dashed TP1 line, half-and-half management.
  - **Arrow-Way / Smart Exit** on opposite signal (yellow `exit` marker).
  - **Backtest table** — W/L, win-rate, P/L, Profit Factor with BUY/SELL × normal/strong breakdown and an OUTCOMES section (TP1+TP2 / TP1+SL / SL / Exit-BE), mirroring SPECTRA's layout.
- **v1.1.1** — first compile pass in the Pine Editor:
  - Fixed `CE10235` in the FVG manager (an if/else where one branch returned `box` and the other `void`) — reordered so both branches end on a void call.
  - Shortened `shorttitle` to `SENTINEL` (was 11 chars; the limit is 10).
- **v1.1.0** — folded in lessons from the SPECTRA setup video ([`docs/spectra-setup-notes.md`](docs/spectra-setup-notes.md)):
  - **No-repaint mode** (`Lock signals at candle close`, default ON) — signals/alerts commit only on the confirmed candle.
  - **Momentum (RSI) + volume confirmation filters** — added as confluences (keep both ON); confluence max scales with which filters are enabled (`X/maxConf`).
  - **Normal vs STRONG signals** — separate `strongConf` threshold upgrades high-conviction setups.
  - **Alert routing** — All / Buy only / Sell only / Strong only.
  - Dashboard gains Momentum, Volume and live Signal rows.
- **v1.0.1** — refinements after first review:
  - FVG confluence now requires price to be **interacting with a live FVG zone**, not merely that one exists somewhere on the chart.
  - Liquidity sweeps fire **once per fresh sweep** (no repeat while the wick stays beyond the swing) — applies to labels, signals and alerts.
  - Sweep *detection* decoupled from the *display* toggle, so hiding sweep labels no longer silently drops the sweep confluence.
  - Note: hiding FVGs (`Show FVGs` off) disables the FVG confluence, since zones are only tracked when drawn.
- **v1.0.0** — initial Foundation release.

## Saving tuned configs (presets)

Because a code-baked preset would freeze the overridden inputs, use TradingView's
**native** mechanisms instead — every input stays live and editable:

- **Save as Default** — tune the inputs, then in the settings dialog click the
  **"Defaults"** dropdown (bottom-left) → **Save as Default**. New instances load
  with your tuned values.
- **Per-symbol chart layouts** — save a separate chart layout per instrument
  (e.g. one for USTEC, one for XAUUSD); each remembers its own SENTINEL settings.
- **Indicator template** — save SENTINEL (with settings) as a template from the
  templates icon on the toolbar, then apply it to any chart.

### Tuned configs (record)

**USTEC / NAS100 · 5m — LOCKED** (≈ Profit Factor 1.41 in testing). Full input list:

| Section | Input | Value |
|---|---|---|
| HTF Bias | Bias EMA length | 25 |
| | HTF 1 / 2 / 3 | 1D / 4H / 30m |
| Structure | Swing pivot length | 8 |
| FVG | Max FVGs / delete when mitigated | 6 / on |
| Sessions | Timezone | America/New_York |
| | Asian / London / NY kill zones | 20:00–00:00 / 02:00–05:00 / 08:30–11:00 |
| Confirmation | RSI length / threshold | 20 / 50 |
| | Volume average length / multiple | 21 / 1 |
| | Kalman (process / measurement noise) | on (0.01 / 0.10) |
| Signals | Trade direction | Both |
| | Min confluence — long / short / STRONG | 5 / 4 / 5 |
| | Lock signals at candle close | on |
| Trade mgmt | Stop-loss S/R lookback | 15 |
| | ATR length / stop buffer | 14 / 0.5 |
| | TP1 / TP2 (R) | 1 / 3.2 |
| | Arrow-Way exit | on |
| Order Blocks | OB search lookback / max per side | 16 / 5 |
| Overlays | Reversal Zones / length / deviation | on / 30 / 2 |

**Risk (strategy):** 1.5% per trade on $50k. **Costs:** USTEC on IC Markets is
spread-only (~0.9 pt). Symbol info: **tick size 0.01, point value 1** → 1 pt = 100
ticks, so the ~0.9 pt spread ≈ **45 ticks slippage per fill** (commission 0).

**USTEC / NAS100 · 20m — ✅ 12-YEAR VALIDATED (definitive config).**
Costed strategy tester (DEEP, 2014–2026, **2,804 trades**): **+263.6%, Profit
Factor 1.149, max DD 17.4%, win 36.1%.** Large-sample, survived 2018 / 2020 / 2022 —
take **PF ~1.15 as the durable edge** (the 3-yr window read higher at ~1.39, but
that's a favourable recent stretch).

| Section | Input | Value |
|---|---|---|
| Risk | Risk per trade / compound | 1.8% / off |
| HTF Bias | Bias EMA length | 101 |
| | HTF 1 / 2 / 3 | 1M / 2W / 1W |
| Structure | Swing pivot length | 5 |
| Confirmation | RSI filter / length / threshold | on / 13 / 50 |
| | Volume filter | **off** |
| | Kalman | on (0.01 / 0.10) |
| Signals | Min confluence — long / short / STRONG | 4 / 4 / 5 |
| Trade mgmt | Stop-loss S/R lookback | 15 |
| | ATR length / stop buffer | 14 / 0.5 |
| | TP1 / TP2 (R) | 1.6 / 3.2 |
| Exit | Mode | Fixed TP1/TP2 |
| Filters | HTF bias alignment / kill-zones / min ADX | **on** / off / off |

⚠️ With HTF-align ON and monthly/2-week/weekly bias, this is essentially a
**trend-follower aligned to NASDAQ's secular uptrend** — a chunk of the edge is
drift capture, so expect degradation in a prolonged bear regime. Costs: slippage
45 ticks, commission 0, margin 1%.

> Both scripts now **default to this 12-year config** — "Reset settings to defaults"
> loads it one-click. (Strategy-only inputs — risk 1.8%, HTF-align filter ON — live
> in the strategy file; the indicator carries the shared signal values.)

### Forward-test results (USTEC 5m, strategy, DEEP)

⚠️ **Reality check.** The on-chart indicator PF (1.41) was tuned on a short recent
window. The costed `strategy()` over longer windows tells a different story:

| Window | PnL | Profit Factor | Win% | Max DD |
|---|---|---|---|---|
| 365d | −0.70% | **0.999** | 41.9% | **47.3%** |
| 90d | +22.1% | 1.144 | 44.4% | 12.4% |
| 30d | −0.67% | 0.985 | 42.7% | 15.7% |
| 7d | −5.14% | 0.759 | 33.3% | 12.2% |

(These ran with ~zero slippage; real 0.9 pt spread makes them worse.) **Conclusion:
not a deployable mechanical edge as-is** — full-year PF ≈ breakeven with a ~47%
drawdown. Next: re-tune for *robustness* (optimise over a long window, validate
out-of-sample / walk-forward), prioritise cutting drawdown, target PF >1.2
consistently across windows with max DD <~20%. Treat the indicator as
decision-support, not a turnkey system, until that bar is cleared.

> Note: TradingView backtest numbers shift between sessions with the amount of
> loaded history — compare configs with the same bars loaded.

## Install

1. Open **TradingView → Chart → Pine Editor**.
2. Open the contents of [`SENTINEL_v1.pine`](SENTINEL_v1.pine), paste into the editor.
3. Click **Save**, then **Add to chart**.
4. Tune inputs via the gear icon. Sensible starting points:
   - **Swing pivot length** `8` (raise for higher TFs / less noise).
   - **Min confluence** `3` (raise to `4`–`5` for stricter, fewer signals).
   - **Session timezone** `America/New_York` (ICT kill zones are defined in NY time).

> The script targets **Pine Script v6**. It is written to spec but has not been
> machine-compiled in this repo — do a final compile-check in the Pine Editor
> (it flags any line instantly). Report any error text and it'll be fixed.

### Alerts

- Use **dynamic alerts**: create an alert on the indicator, condition `Any alert() function call` — messages include symbol, side, confluence and price.
- Or use the **template conditions** dropdown: *SENTINEL Long*, *SENTINEL Short*, *SENTINEL Liquidity Sweep*.

---

## Design notes / honest scoping

The full vision (12 modules: order blocks, confidence grading, dynamic trade manager,
backtester, Kalman trend, regime classifier, multi-symbol scanner) is realistically
3,000–8,000+ lines of Pine and pushes TradingView's per-script limits (objects, execution
time, memory). Trying to generate all of it in one pass produces subtly buggy code.

So it's built **in phases**, each compilable and testable before the next is added:

- **v2 — Execution:** ✅ done — trade manager + backtest table + Arrow-Way exit.
- **v3 — Professional:** ✅ done — order-block engine, confidence grade (A+/A/B), Smart Trail / Trend Cloud / Reversal Zone overlays.
- **v4 — Elite:** ✅ done — Kalman trend filter, market-regime classifier, POI ranking, multi-symbol scanner.
- **Beyond v4 (future ideas):** multi-timeframe confluence matrix, volume-profile / HVN integration, cumulative delta where available, replay/journaling hooks, off-platform (ML-assisted) parameter optimisation.

---

## Repo layout

```
sentinel/
├── SENTINEL_v1.pine        # the indicator — visuals, dashboard, signals (paste as Indicator)
├── SENTINEL_strategy.pine  # the strategy — same signals, executes via strategy.* (paste as Strategy)
├── README.md               # this file
└── docs/
    ├── background.md           # the SentioEdge/SPECTRA review that motivated the design
    └── spectra-setup-notes.md  # SPECTRA setup-video notes used as design input
```

## Forward testing

Use `SENTINEL_strategy.pine` for out-of-sample testing — it mirrors the
indicator's signal engine but runs through TradingView's **Strategy Tester**
(realistic fills, commission, slippage) and keeps updating as new live bars
arrive (a rolling forward test once inputs are locked).

1. Pine Editor → paste `SENTINEL_strategy.pine` → **Add to chart** (it loads as a strategy).
2. Strategy Tester tab → read PF, win-rate, max drawdown, equity curve.
3. **Properties** tab → set commission & slippage to your broker (USTEC ≈ spread-only, ~0.9 pt).
4. For paper/live: create an alert on the strategy → condition **"Any alert() function call"** → point the webhook at your paper broker.

Execution model: 1.5% risk per trade (full-stop loss = 1.5% of the risk base),
½ closes at TP1, ½ runs to TP2 (shared stop), opposite signal reverses
(Arrow-Way), orders fill on bar close (non-repainting).

### Robustness & exit options (strategy, group 8)

For attacking the breakeven/47%-drawdown problem found in testing:
- **Exit mode** — *Fixed TP1/TP2* (default) or **Smart Trail** (½ at TP1, the rest
  rides an ATR trailing stop so winners run and roll over on reversal).
- **Filters** (all default off) — *require HTF bias alignment* (no counter-trend),
  *trade only in kill zones*, *min ADX* (skip range/chop; try 18–22).
- **Risk caps** (default off) — *daily loss limit %* and *max drawdown halt %*
  (built on `strategy.risk.*`) to put a hard ceiling on drawdown.

Recommended first experiment: turn on HTF-align + kill-zone + min-ADX (fewer,
higher-quality trades), test Fixed vs Smart Trail, and add a daily-loss cap.

### Testing other timeframes

Just change the chart timeframe — the strategy runs on it. **Set the HTF inputs
higher than the chart** (rule: all three HTFs ≥ chart TF):

| Chart | HTF 1 / 2 / 3 |
|---|---|
| 5m  | D / 4H / 30m |
| 20m | D / 4H / 1H |
| 1H  | W / D / 4H |

Higher timeframes make the ~0.9 pt spread a much smaller % of each move, so the
costed edge often improves — re-tune per timeframe and use multi-year DEEP history.
