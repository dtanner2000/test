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
| **v1**  | Foundation   | ✅ **Shipped** (`SENTINEL_v1.pine`) |
| v2      | Execution    | ⏳ planned |
| v3      | Professional | ⏳ planned |
| v4      | Elite        | ⏳ planned |

### v1 — Foundation (what's in the box)

1. **Higher-Timeframe Bias** — Daily / 4H / 2H, each scored bull/bear/flat vs an EMA, summed to a net bias (−3…+3).
2. **Fair Value Gaps** — automatic bullish/bearish FVG detection, drawn as boxes, with a full mitigation lifecycle (extend → consume → delete/grey-out).
3. **Liquidity engine** — sweep / stop-hunt detection (wick takes prior swing, closes back inside).
4. **Market structure** — BOS / CHOCH derived from swing pivots, with a running trend state.
5. **Session intelligence** — Asian / London / New York kill-zone shading + PDH/PDL and PWH/PWL liquidity levels.
6. **Confluence scoring** — 5-point long/short score → triangle signals, dynamic `alert()` messages, template `alertcondition()`s, and a compact on-chart dashboard.

---

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

- **v2 — Execution:** order-block engine (bull/bear OBs, breakers, mitigation blocks), POI ranking, confidence grade (A+/A/B…), dynamic trade manager (entry / stop / TP1-3 / RR / position size), statistics panel.
- **v3 — Professional:** Kalman trend filter, adaptive market-regime classifier (trend / pullback / range / breakout / reversal), multi-symbol scanner, deeper performance analytics.
- **v4 — Elite:** multi-timeframe confluence matrix, volume-profile / HVN integration, cumulative delta where available, replay/journaling hooks, off-platform parameter optimisation.

---

## Repo layout

```
sentinel/
├── SENTINEL_v1.pine     # Foundation release — paste into TradingView
├── README.md            # this file
└── docs/
    └── background.md     # the SentioEdge/SPECTRA review that motivated the design
```
