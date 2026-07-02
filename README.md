# Kalshi × Polymarket — FOMC Sep 2026 Strategies

This repo documents 2 prediction-market bots that price the *same* September 2026 FOMC rate decision from different angles. **Strat1** is a cross-venue arbitrage monitor across **Kalshi and Polymarket**: it maps the 5 FOMC outcome buckets, models venue-specific price-dependent fees, and flags a strict 2-leg YES/NO lock that pays \$1 at resolution. **Strat2** is a single-venue **Frank–Wolfe / Bregman** fair-value anchor on Polymarket: it prices the Sep-cut market against a liquidity-weighted blend of related markets and only signals when its 4 confirmation factors agree. The full write-up, with charts and a plain-English appendix, is in [`Kalshi-Polymarket-FOMC-Sep26-strats.pdf`](./Kalshi-Polymarket-FOMC-Sep26-strats.pdf).

Through to 1Jul26 the takeaway is that both venues were tightly aligned, so real cross-venue dislocations were small and rare. Strat1 (47,441 live cycles) has so far locked 5 thin paper arbs for **+\$36.40** on a \$10k book — a real but tiny edge — while its eye-catching 7% spikes proved to be un-fillable thin/dislocated-book artifacts on penny-priced tail legs. Strat2 (37,562 cycles) sat at **WATCH** the entire window: its Bregman edge hovered a mildly-positive ~+30 bp but its confirmation gate never opened, so it opened zero positions. The value here isn't large alpha — it's two verified, reproducible measurement pipelines and a clear, honest failure boundary: the cross-venue arb is the thin real edge, and the anchor stays monitor-only until it proves predictive out-of-sample.

## How to reproduce

Both bots poll their venues on a ~15s cadence and append one JSON record per cycle to a local log. Strat1's paper-sim is a deterministic replay of that log through the executable gate: `forward_test.py --since <ISO-timestamp>` reconstructs the locked-arb track record for any window. The report's figures and metrics are computed directly from those logs for the 20 Jun → 1 Jul 2026 window.

## Data assumptions

Prices are top-of-book from the Kalshi REST/WebSocket API and the Polymarket CLOB. Fees are modelled per leg and price-dependent: Kalshi `0.07·P·(1−P)` and Polymarket `0.05·min(P,1−P)` — the taker rates the monitor logged for these FOMC decision markets — plus `0.001`/leg slippage. A raw edge is only counted as an *executable* opportunity when it clears the gate: a **20–500 bp** net edge with a **<10¢** spread; edges above 500 bp are flagged as thin/dislocated-book artifacts and rejected.

## Known limitations

Top-of-book **sizes are not logged**, so paper fills are depth-capped by a fixed notional rather than by true book depth — the fill model is deliberately simplified. The headline PnL is the sum of 5 independent paper arbs — one per outcome bucket — each sized at **\$1,000** of the \$10,000 book, so **\$5,000 (50%)** was deployed across the 5 legs and \$5,000 sat idle. Each bucket is locked once and held to resolution, so the total is sensitive to the window start; the annualized figures normalize a short 11-day window and are **not** strategy-capacity estimates. Strat2's "edge" is a Bregman (information-geometry) divergence, not a raw price gap.
