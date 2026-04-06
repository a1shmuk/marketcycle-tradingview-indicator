\# MarketCycle Trading Strategy \[BOS \+ Retracement \+ S/R\]

\!\[TradingView\](https://img.shields.io/badge/TradingView-Pine%20Script-blue)  
\!\[Version\](https://img.shields.io/badge/version-1.0-green)  
\!\[License\](https://img.shields.io/badge/license-MIT-yellow)

A comprehensive TradingView Pine Script indicator combining three core price action concepts: \*\*Break of Structure (BOS)\*\*, \*\*2-Candle Retracement Patterns\*\*, and \*\*Support/Resistance\*\* levels with \*\*Supply/Demand zones\*\*.

\#\# 📸 Screenshot  
\*Add your screenshot here\*

\#\# 🎯 Features

\#\#\# 1\. Strong Trend Detection  
\- Identifies strong candles (configurable body percentage threshold)  
\- Green triangles (▲) for strong bullish candles  
\- Red triangles (▼) for strong bearish candles

\#\#\# 2\. 2-Candle Retracement Pattern  
\- Detects pullback entries in trending markets  
\- "2-CR" labels for optimal entry points  
\- Filters for minimum candle body size

\#\#\# 3\. Break of Structure (BOS)  
\- Automatic bullish/bearish structure breaks detection  
\- Green circles for bullish BOS (break above highs)  
\- Red circles for bearish BOS (break below lows)  
\- Visual Supply/Demand zones extending forward

\#\#\# 4\. Support & Resistance  
\- Dynamic pivot-based levels  
\- Auto-updating lines (keeps last 5 levels)  
\- Green lines \= Resistance, Red lines \= Support

\#\# 🚀 Installation

1\. Open TradingView.com  
2\. Open Pine Editor (bottom panel)  
3\. Create New Indicator  
4\. Paste the code from \`marketcycle.pine\`  
5\. Click "Add to Chart"

\#\# ⚙️ Settings

| Setting | Default | Description |  
|---------|---------|-------------|  
| Strong Candle Body % | 60% | Minimum body size for strong candle |  
| BOS Lookback | 10 bars | Period for structure detection |  
| Extend Zones | 20 bars | How far S/D zones extend |  
| Pivot Lookback | 10 bars | S/R pivot detection sensitivity |

\#\# 📊 How to Use

\*\*Bullish Setup:\*\*  
1\. Look for green "BOS" circle (structure break)  
2\. Wait for price to return to green demand zone  
3\. Look for "2-CR" label (retracement complete)  
4\. Entry on strong bullish candle

\*\*Bearish Setup:\*\*  
1\. Look for red "BOS" circle  
2\. Wait for price to return to red supply zone    
3\. Look for orange "2-CR" label  
4\. Entry on strong bearish candle

\#\# 🔔 Alerts  
\- Bullish/Bearish BOS detection  
\- 2-Candle Retracement patterns

\#\# 👥 Credits  
Developed as a Minor Project implementing MarketCycle trading methodology.

\#\# 📄 License  
MIT License \- Feel free to modify and distribute.  
