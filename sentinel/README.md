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
├── SENTINEL_v1.pine     # Foundation release — paste into TradingView
├── README.md            # this file
└── docs/
    └── background.md     # the SentioEdge/SPECTRA review that motivated the design
```
