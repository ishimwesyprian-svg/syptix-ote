# Syptix OTE Trading System — Master Specification

**Version:** 0.1  
**Status:** Initial design  
**Purpose:** Define the trading rules and product requirements for Syptix before implementation.

> This document separates confirmed strategy rules from decisions that still require explicit approval. The implementation must not silently invent rules that change the strategy.

## 1. Product vision

Syptix is a configurable trading application for market-structure analysis, OTE-based trade setup detection, visual historical backtesting, analytics, and eventually paper/live execution through a separately controlled MT5 integration.

The application should explain why a setup was created, updated, entered, skipped, or invalidated.

## 2. Timeframe configuration

- The user can select the higher timeframe (HTF) and lower timeframe (LTF) in the UI.
- An optional adaptive timeframe mode may recommend an LTF based on the selected HTF.
- Manual selection remains available.
- The selected timeframes must be used consistently in the strategy engine, charts, backtester, and logs.
- The UI must validate timeframe selections and explain any incompatible configuration.

## 3. Market-structure engine

Implement the user's existing definitions for:
- Valid swing highs and swing lows.
- Valid pullbacks.
- Valid break of structure (BOS).
- Protected swings.
- Change of Character (CHOCH).

### BOS
A BOS must satisfy the strategy's liquidity and structure requirements. Where the strategy requires a candle close beyond a swing, a wick-only break is not a valid BOS.

### CHOCH
A CHOCH is confirmed when a fully closed candle on the relevant timeframe closes beyond the protected swing identified by the strategy. A wick-only break does not qualify.

The system must distinguish a confirmed structural change from a fully established new trend. Use the original swing-validation rules consistently on both HTF and LTF charts.

## 4. HTF Fibonacci and OTE

After a valid HTF BOS:

1. Identify the relevant impulse leg using the strategy's swing-selection rules.
2. For a bullish structure, identify the qualifying lowest swing low in the impulse leg and the immediate new valid swing high formed after the break.
3. For a bearish structure, apply the inverse logic.
4. Draw the HTF Fibonacci across the selected range.
5. Identify the conventional ICT OTE zone, approximately the 62%–79% retracement, with 70.5% as a reference level.
6. Identify and display the HTF target zone.
7. Monitor price for entry into the OTE zone.

Fibonacci orientation and price-level calculations must be explicit and separately tested for bullish and bearish setups.

## 5. HTF structure replacement

If price does not reach the active HTF OTE zone and a new valid HTF structure supersedes the existing setup:

1. Abandon the obsolete setup.
2. Recalculate the relevant impulse leg.
3. Draw a new Fibonacci.
4. Identify the new OTE zone and target.
5. Monitor the new setup.

A newly formed swing alone must not automatically invalidate an existing setup. The new structure must satisfy the strategy's validity criteria. The exact supersession rule remains an implementation decision to confirm against the user's original structure rules.

## 6. LTF confirmation

When price reaches the active HTF OTE zone:

1. Activate monitoring on the selected LTF.
2. Identify the relevant protected swing.
3. Wait for a valid LTF CHOCH in the direction of the HTF bias.
4. Identify the qualifying LTF impulse leg using the original swing and BOS rules.
5. Draw the LTF Fibonacci.
6. Mark its 50% equilibrium level.

## 7. Entry rules

After a valid LTF CHOCH, wait for price to reach the 50% equilibrium of the relevant LTF Fibonacci. A CHOCH alone is not an entry signal.

If price never reaches equilibrium:
- Do not chase price.
- Wait for the next qualifying LTF structure in the direction of the active HTF bias.
- Use the original swing, BOS, and impulse-leg rules to determine the qualifying structure.
- Redraw the LTF Fibonacci and evaluate its 50% equilibrium.
- Repeat while the HTF setup remains valid and its target has not been reached.

## 8. Stop-loss rules

- Bullish setup: stop loss below the relevant LTF Fibonacci 100% anchor.
- Bearish setup: stop loss above the relevant LTF Fibonacci 100% anchor.

The stop buffer, spread allowance, minimum stop distance, and broker-specific execution constraints must be explicitly defined before live execution.

## 9. Take-profit rules

The take-profit objective is the selected HTF target high or low. The target-selection rule still needs to be defined precisely.

Do not silently substitute a fixed risk-to-reward target for the selected HTF target.

## 10. HTF CHOCH and invalidation

If an opposing HTF CHOCH occurs before entry:

1. Cancel the conflicting pending setup.
2. Stop evaluating entry opportunities tied to the invalidated structure.
3. Record the CHOCH and the broken protected swing.
4. Reassess the market structure.
5. Evaluate new opportunities using the OTE framework.

If an opposing HTF CHOCH occurs while a trade is open, apply the configured open-position management rule. A CHOCH must not automatically close an open position unless that behavior is explicitly selected.

## 11. HTF target reached

When the selected HTF target zone is reached, stop searching for additional entries associated with that HTF setup. Record the event and display it on the chart.

The system must distinguish target completion from structural invalidation.

## 12. User interface

Use a clean, premium financial-dashboard design with these planned modules:
- Overview
- Markets
- Strategy & Structure
- Positions
- Backtester
- Analytics
- Logs
- Settings

The strategy panel must expose HTF and LTF selection and relevant configurable parameters.

Chart annotations should include:
- Valid swing highs and lows.
- BOS and CHOCH markers.
- Protected swings.
- HTF Fibonacci and shaded OTE zone.
- HTF target zone.
- LTF Fibonacci and equilibrium.
- Entry, stop-loss, and take-profit levels.
- Active, replaced, missed, and invalidated setup states.

The application should explain skipped entries and invalidated setups.

## 13. Visual backtester

The backtester should support historical data and graphical replay, and report:
- Number of trades.
- Winning and losing trades.
- Win rate.
- Gross and net profit/loss.
- Drawdown.
- Risk/reward statistics.
- Equity curve.
- Trade duration.
- Trades by day, week, and month.
- Setup invalidations and missed entries.

Prevent look-ahead bias: a swing should only become available after enough candles have formed to confirm it. Simulated execution should account for defined trading costs and execution assumptions.

## 14. Development stages

1. Finalize the strategy specification.
2. Define the software architecture and repository.
3. Implement and test market-structure calculations.
4. Implement HTF OTE and LTF entry engines.
5. Build the UI and historical backtester.
6. Validate results against manually inspected chart examples.
7. Add paper trading.
8. Develop and test MT5 integration separately from simulation.

Live trading should remain disabled until strategy logic, risk controls, and execution are tested. No profitability is guaranteed.

## 15. Decisions still requiring confirmation

1. Exact HTF target-selection rule.
2. Whether the LTF equilibrium entry requires an exact touch or configurable tolerance.
3. What happens to an open trade after an opposing HTF CHOCH.
4. Position sizing and maximum-risk controls.
5. Rules for simultaneous setups and multiple open positions.
6. Data provider, supported symbols, and data limitations.
7. Spread, commission, slippage, and execution assumptions.
8. Exact criteria for a superseding HTF structure.
9. Handling of repeated LTF CHOCH events.
10. Handling of gaps through entry, stop-loss, or target levels.

Unresolved decisions must be surfaced to the user and must not be silently filled with assumptions that alter the strategy.
