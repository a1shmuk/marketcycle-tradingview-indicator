# SMC Visual Bot — Pine Script v5

A professional Smart Money Concepts indicator suite for TradingView.
Built for teaching, signal generation, and visual price action analysis.

---

## Indicators Included

### 1. SMC Visual Bot (v6)
**File:** `indicators/SMC_Visual_Bot_v6.pine`

Signal-focused visual indicator. No clutter, no SL/TP lines.

- EMA200 trend filter — bull zones above EMA, bear zones below
- 3-candle FVG detection (Fair Value Gap)
- 3-candle OB detection (Order Block)
- Zone invalidation — box auto-deletes when price closes through it
- Max 2 active zones per direction (configurable)
- Re-entry signal — fires only when price returns to a zone
- One signal per zone (fired guard prevents repeats)
- Auto-adapts parameters to timeframe (scalp / intraday / swing)
- Dashboard: live trend, EMA value, zone count, signal count
- alertcondition() for BUY and SELL

---

### 2. SMC Education Toolkit (v2)
**File:** `indicators/SMC_Education_Toolkit_v2.pine`

Teaching-focused. Every concept labelled and explained in comments.

- Layer 1: Swing H/L labels, BOS detection, trend via HH/HL structure
- Layer 2: FVG boxes, Order Blocks, BSL/SSL liquidity dotted lines, Premium/Discount zones
- Layer 3: Buy/Sell signals with alignment check, dashboard

**Key fix in v2:** Zone boxes now anchor to the correct swing pair (min of highBar, lowBar) instead of the most recent pivot independently.

---

### 3. MarketCycle Pro
**File:** `indicators/MarketCycle_Pro.pine`

Strategy-focused with BOS + CHoCH + retracement pattern detection.

- Filtered BOS: cooldown + minimum move % to eliminate noise
- CHoCH (Change of Character): trend reversal detection in purple
- 2-Candle Retracement (2-CR): entry timing with cooldown guard
- Supply/Demand zones labelled as BOS or CHoCH origin
- Support/Resistance: pivot-based horizontal lines (max 5 each)
- Strong candle filter: body % threshold
- Info panel: trend, last BOS/CHoCH, cooldown counters

**Key fix:** Consecutive 2-CR bug — signals now use `lastFiredBar` guard, minimum 2-bar gap between triggers.

---

## Settings Guide (XAUUSD 1H)

| Setting | Recommended |
|---|---|
| Swing Length | 7–10 |
| Max Active Zones | 2 |
| OB Impulse Multiplier | 1.5–2.0 |
| EMA Length | 200 |
| BOS Min Move % | 0.3–0.5 |
| 2-CR Cooldown | 5 bars |

---

## How to Install

1. Open TradingView → Pine Editor
2. Click "New indicator"
3. Select All → Delete → Paste the `.pine` file content
4. Save → Add to chart

---

## Changelog

See `docs/CHANGELOG.md`

---

## Disclaimer

For educational purposes only. Trading involves risk.
