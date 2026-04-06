// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/  
// © MarketCycle

//@version=5  
indicator("MarketCycle Complete Strategy \[BOS \+ Retracement \+ S/R\]",   
     overlay=true,   
     max\_lines\_count=500,   
     max\_boxes\_count=500,  
     max\_labels\_count=500)

// ═════════════════════════════════════════════════════════════════════════════  
// INPUTS  
// ═════════════════════════════════════════════════════════════════════════════

// Trend Strength Settings  
strongCandleThreshold \= input.float(60, "Strong Candle Body %", minval=10, maxval=100, group="Trend Settings")  
trendLookback \= input.int(5, "Trend Lookback Period", minval=2, maxval=20, group="Trend Settings")

// BOS Settings  
bosLookback \= input.int(10, "BOS Lookback Period", minval=5, maxval=50, group="BOS Settings")  
showBullishBOS \= input.bool(true, "Show Bullish BOS", group="BOS Settings")  
showBearishBOS \= input.bool(true, "Show Bearish BOS", group="BOS Settings")

// Supply/Demand Zones  
extendZones \= input.int(20, "Extend Zones (bars)", minval=5, maxval=100, group="Supply/Demand Settings")  
zoneHeight \= input.float(0.5, "Zone Height Multiplier", minval=0.1, maxval=2.0, step=0.1, group="Supply/Demand Settings")

// Support/Resistance  
pivotLookback \= input.int(10, "Pivot Lookback", minval=5, maxval=50, group="Support/Resistance")  
showSRLevels \= input.bool(true, "Show S/R Levels", group="Support/Resistance")

// Retracement Settings  
retraceCandle1Size \= input.float(50, "Retracement Candle 1 Min Body %", minval=20, maxval=100, group="Retracement Pattern")  
retraceCandle2Size \= input.float(50, "Retracement Candle 2 Min Body %", minval=20, maxval=100, group="Retracement Pattern")

// ═════════════════════════════════════════════════════════════════════════════  
// CALCULATIONS  
// ═════════════════════════════════════════════════════════════════════════════

// Candle Properties  
bodySize \= math.abs(close \- open)  
candleRange \= high \- low  
bodyPercent \= (bodySize / candleRange) \* 100  
isStrongCandle \= bodyPercent \>= strongCandleThreshold  
isBullish \= close \> open  
isBearish \= close \< open

// Trend Detection  
higherHighs \= high \> ta.highest(high\[1\], trendLookback)  
lowerLows \= low \< ta.lowest(low\[1\], trendLookback)

// ═════════════════════════════════════════════════════════════════════════════  
// 2-CANDLE RETRACEMENT PATTERN DETECTION  
// ═════════════════════════════════════════════════════════════════════════════

var bool inBullishTrend \= false  
if higherHighs  
    inBullishTrend := true  
if lowerLows and inBullishTrend\[1\]  
    inBullishTrend := false

var bool inBearishTrend \= false  
if lowerLows  
    inBearishTrend := true  
if higherHighs and inBearishTrend\[1\]  
    inBearishTrend := false

// Pattern Detection  
candle1Bearish \= isBearish and (bodyPercent \>= retraceCandle1Size)  
candle2Bearish \= isBearish and (bodyPercent \>= retraceCandle2Size)  
candle1Bullish \= isBullish and (bodyPercent \>= retraceCandle1Size)  
candle2Bullish \= isBullish and (bodyPercent \>= retraceCandle2Size)

bearishRetracement \= inBullishTrend\[2\] and candle1Bearish\[1\] and candle2Bearish  
bullishRetracement \= inBearishTrend\[2\] and candle1Bullish\[1\] and candle2Bullish

// ═════════════════════════════════════════════════════════════════════════════  
// BOS (BREAK OF STRUCTURE) DETECTION  
// ═════════════════════════════════════════════════════════════════════════════

var float lastBullHigh \= 0  
var float lastBullLow \= 0  
var int lastBullBar \= 0

if high \> ta.highest(high\[1\], bosLookback)  
    lastBullHigh := high  
    lastBullLow := low  
    lastBullBar := bar\_index

var float lastBearLow \= 0  
var float lastBearHigh \= 0  
var int lastBearBar \= 0

if low \< ta.lowest(low\[1\], bosLookback)  
    lastBearLow := low  
    lastBearHigh := high  
    lastBearBar := bar\_index

bullishBOS \= bar\_index \== lastBullBar  
bearishBOS \= bar\_index \== lastBearBar

// ═════════════════════════════════════════════════════════════════════════════  
// SUPPLY & DEMAND ZONES  
// ═════════════════════════════════════════════════════════════════════════════

var box demandZone \= na  
if bullishBOS  
    if not na(demandZone)  
        box.delete(demandZone)  
    zoneTop \= lastBullHigh  
    zoneBottom \= lastBullLow  
    demandZone := box.new(left=bar\_index, top=zoneTop, right=bar\_index+extendZones, bottom=zoneBottom,   
         bgcolor=color.new(color.green, 85), border\_color=color.green, border\_width=1)

var box supplyZone \= na  
if bearishBOS  
    if not na(supplyZone)  
        box.delete(supplyZone)  
    zoneTop \= lastBearHigh  
    zoneBottom \= lastBearLow  
    supplyZone := box.new(left=bar\_index, top=zoneTop, right=bar\_index+extendZones, bottom=zoneBottom,   
         bgcolor=color.new(color.red, 85), border\_color=color.red, border\_width=1)

if not na(demandZone)  
    box.set\_right(demandZone, bar\_index \+ extendZones)  
if not na(supplyZone)  
    box.set\_right(supplyZone, bar\_index \+ extendZones)

// ═════════════════════════════════════════════════════════════════════════════  
// SUPPORT & RESISTANCE LEVELS  
// ═════════════════════════════════════════════════════════════════════════════

pivotHighVal \= ta.pivothigh(high, pivotLookback, pivotLookback)  
pivotLowVal \= ta.pivotlow(low, pivotLookback, pivotLookback)

var line\[\] resistanceLines \= array.new\_line(0)  
var line\[\] supportLines \= array.new\_line(0)

if not na(pivotHighVal) and showSRLevels  
    newLine \= line.new(bar\_index \- pivotLookback, pivotHighVal, bar\_index \+ 20, pivotHighVal,   
         color=color.green, width=2, style=line.style\_solid)  
    array.push(resistanceLines, newLine)  
    if array.size(resistanceLines) \> 5  
        line.delete(array.shift(resistanceLines))

if not na(pivotLowVal) and showSRLevels  
    newLine \= line.new(bar\_index \- pivotLookback, pivotLowVal, bar\_index \+ 20, pivotLowVal,   
         color=color.red, width=2, style=line.style\_solid)  
    array.push(supportLines, newLine)  
    if array.size(supportLines) \> 5  
        line.delete(array.shift(supportLines))

// ═════════════════════════════════════════════════════════════════════════════  
// VISUALIZATION  
// ═════════════════════════════════════════════════════════════════════════════

// Strong Candle Highlighting  
plotshape(isStrongCandle and isBullish, "Strong Bullish Candle", shape.triangleup, location.belowbar, color.green, size=size.small)  
plotshape(isStrongCandle and isBearish, "Strong Bearish Candle", shape.triangledown, location.abovebar, color.red, size=size.small)

// 2-Candle Retracement Patterns  
plotshape(bullishRetracement, "Bullish Retracement Entry", shape.labelup, location.belowbar, color.lime,   
     text="2-CR", textcolor=color.white, size=size.normal)  
plotshape(bearishRetracement, "Bearish Retracement Entry", shape.labeldown, location.abovebar, color.orange,   
     text="2-CR", textcolor=color.white, size=size.normal)

// BOS Signals  
plotshape(bullishBOS and showBullishBOS, "Bullish BOS", shape.circle, location.belowbar, color.green,   
     text="BOS", textcolor=color.green, size=size.small)  
plotshape(bearishBOS and showBearishBOS, "Bearish BOS", shape.circle, location.abovebar, color.red,   
     text="BOS", textcolor=color.red, size=size.small)

// Background coloring for trend context  
bgcolor(inBullishTrend ? color.new(color.green, 95\) : inBearishTrend ? color.new(color.red, 95\) : na)

// ═════════════════════════════════════════════════════════════════════════════  
// ALERTS  
// ═════════════════════════════════════════════════════════════════════════════

alertcondition(bullishBOS, "Bullish BOS", "Bullish Break of Structure Detected\!")  
alertcondition(bearishBOS, "Bearish BOS", "Bearish Break of Structure Detected\!")  
alertcondition(bullishRetracement, "Bullish Retracement", "2-Candle Bullish Retracement Pattern")  
alertcondition(bearishRetracement, "Bearish Retracement", "2-Candle Bearish Retracement Pattern")  
