# Claude Code Model Fable 5 UltraCode Proposed Solution 

## Proposed solution after reviewing the origin takehome and live trading history

before any action, please thoroughly evaluate your approach, compared to pattern selection, probability modeling, etc with your agent teams


Ran 4 agents
All four evaluation agents are now running in parallel against the dataset. What each must deliver — numbers, not opinions:

Track	Core question	Kill criterion
Probability modeling	Does Φ(Δ/√(σ²τ+σ²_basis)) beat the market mid's log-loss on validation days? Where's the mispricing map?	Model can't beat market's own calibration → no edge to trade
Lead-lag	After a ≥$15 Binance jump, how many cents of token move remain after t+2s (our earliest fill)?	Residual move < round-trip cost → latency edge is dead
Pattern selection	Do prior-era patterns add log-loss improvement beyond (Δ, τ, σ) on validation? Are pattern coefficients stable across train halves?	No incremental information → patterns were proxies for state variables + noise
ML features	Does LightGBM beat both market mid and the analytic model out-of-sample? Does the ablation show real signal beyond "blend price with delta"?	Train/validation gap large or margin tiny → complexity not earned
All four operate under the frozen-split discipline — fit on Jun 24–29, judge on Jun 30–Jul 1, test days never touched.

Two things the bake-off cannot settle, flagging them now so they don't get lost:

Maker-style execution for real live trading — the tapes only have top-of-book snapshots, no trade prints, so we can't measure fill probabilities for resting limit orders offline. If live trading is the priority, this eventually needs its own data collection (CLOB trade feed). The signal layer being evaluated now feeds either execution style.
Score-profile shaping (Sharpe/drawdown engineering for the harness formula) — that's a policy layer on top of whichever signal wins; I'll analyze it in the synthesis rather than delegate it.
My expectation going in — stated now so you can hold me to it: probability modeling and lead-lag are likely complements (same underlying anchor, different horizons), patterns likely fail the marginality test, and ML likely adds a small margin that isn't worth single-file export risk in v1 but becomes relevant later. If the data says otherwise, the data wins.

I'll synthesize all four reports into a comparison matrix with a recommended architecture when they land — then we discuss before any strategy code gets written.


## Conclusion from Fable's quick test
The coherent picture: Polymarket's 5-min BTC market in mid-2026 is made by professionals pricing off the actual Chainlink feed at sub-second latency. At 1Hz, taker-only, paying 0.5% slippage + 0.072·p(1−p) fees, there is no extractable alpha from public information. The prior project's replay wins (±$100–700/window) were noise fit to noise. This is a strongly-evidenced negative result, and it's worth more than another 30 guard versions.

My recommendation — one cheap, decisive experiment before any strategy decision:
Record both feeds at high frequency for a few hours (Polymarket CLOB WS books + trades channel, Binance WS trades — pure listening, ~100ms resolution, no trading). It answers caveats 1 and 2 directly and produces the fill-rate data a maker strategy would need. One evening of data settles whether:

(a) a sub-second latency edge exists within harness reach → build the sniper around it;
(b) nothing survives even at 100ms → for the take-home, submit an honest defensive model (Φ-model risk layer, near-zero exposure, documented negative-result thesis); for live trading, the only structurally sound path is maker/liquidity provision (earn the spread + rebates that we've been paying), which the same recording bootstraps;
(c) or we conclude the market isn't worth attacking and redirect effort.

## Renamed from 'fable-proposal.md' to 'Fable-Proposal-Conclusion.md'