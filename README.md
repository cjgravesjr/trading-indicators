# trading-indicators

Pine Script (TradingView) indicators and companion tools built and iterated by CJ Graves Jr. for systematic intraday trading on index futures and SPX options. Published on TradingView; mirrored here with change history.

**This is code and tooling, not trading advice.** Nothing in this repository is a recommendation to buy or sell anything.

## What's here

| Folder / file | What it is |
|---|---|
| `liquidity-radar/MBF_RZ_Liquidity_v0_6_12.pine` | **MBF & RZ Liquidity Radar v0.6.12** — maps where resting liquidity sits (swing highs/lows, session extremes, prior day/week, equal highs/lows), detects sweeps, reclaims and rejections with a bar-close state machine, and narrates the live "story" (WAIT / WATCH / LONG / SHORT / RUNNING / DONE) in a panel. Exports five bridge values that the ULTRA indicator reads. Pine v6, ~1,265 lines. |
| `tools/` | Self-contained HTML tools built alongside the indicators (backtest lab, contract-size sheet, top-down game plan checklist, VWAP + stochastics explainer). |

## Liquidity Radar, in plain language

1. **Pools.** The script keeps a list of price levels where stop orders tend to cluster. Each pool knows its source (swing, session, PDH/PDL, PWH/PWL, equal highs/lows), a weight, and a state.
2. **State machine (bar close only).** `untouched -> swept` when a wick pierces the level by at least a minimum depth and the bar closes back; `untouched -> broken` when a bar closes through it; `swept -> reclaimed` when a body closes back through inside a short window; `reclaimed -> rejected` when the level is retested and holds. Running transitions on confirmed bars only was the fix for intrabar "flashing".
3. **Story layer.** A sweep starts a story; a reclaim plus a lower-timeframe structure break arms it; the nearest untouched pool on the story side is the target; the swept wick (then the last confirmed higher-low / lower-high) is the stop. A fresh LONG/SHORT prints only if reward-to-risk clears a minimum and the stop is inside a maximum ATR width.
4. **Trend vote.** Chart structure + higher-timeframe bias + the direction pools are breaking. Two of three agree = a trend; otherwise MIXED.
5. **Bridge exports.** `lq_state`, `lq_age`, `lq_swept_wick`, `nd_level`, `trend_dir` are plotted to the data window so a second indicator (ULTRA) can read them through `input.source`.

### Engineering notes worth reading in the header comments

- O(n²) per-tick draw loop replaced with two linear passes + a sort (v0.5.3) after the chart timed out at the pool cap.
- Redraw throttle: drawing runs on bar close or after N ticks of price movement; detection and alerts are never throttled.
- Story ownership bug (v0.3.2): any pool's reclaim could stamp "reclaimed" onto another pool's story. Fixed by weight-ordered takeover and owner-only reclaim.
- Coherence guard (v0.5.8): the published draw direction must have a pool on the correct side of price, or it prints nothing. "An honest blank beats a confident wrong arrow."
- Storyteller overlap (v0.6.7): Pine tables never wrap, so the narrative line is trimmed and hard-capped.

## Version history

See the changelog block at the top of each `.pine` file. Versions are dated and each entry says what changed, why, and what did not change.

## Author

Clarence (CJ) Graves Jr. — telecom field engineer moving into systems/support analysis. [LinkedIn](https://www.linkedin.com/in/clarence-graves-jr-8035342a0) · [homelab](https://github.com/cjgravesjr/homelab)

"RZ" in the file headers credits a trading partner's contributions to the HTF bias and LTF CHoCH modules.
