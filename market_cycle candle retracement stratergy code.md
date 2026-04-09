//@version=5
indicator("MarketCycle Pro [Filtered BOS + CHoCH + Retracement + S/R]", overlay=true, max_lines_count=500, max_boxes_count=500, max_labels_count=500)

// ═════════════════════════════════════════════════════════════════════════════
// INPUTS
// ═════════════════════════════════════════════════════════════════════════════

// Trend Strength Settings
strongCandleThreshold = input.float(60, "Strong Candle Body %", minval=10, maxval=100, group="Trend Settings")
trendLookback = input.int(5, "Trend Lookback Period", minval=2, maxval=20, group="Trend Settings")

// BOS & CHoCH Settings
bosLookback = input.int(10, "BOS Lookback Period", minval=5, maxval=50, group="BOS/CHoCH Settings")
bosCooldown = input.int(5, "BOS Cooldown (bars)", minval=3, maxval=20, group="BOS/CHoCH Settings")
bosMinMove = input.float(0.5, "BOS Min Move (%)", minval=0.1, maxval=5.0, step=0.1, group="BOS/CHoCH Settings")
showBullishBOS = input.bool(true, "Show Bullish BOS", group="BOS/CHoCH Settings")
showBearishBOS = input.bool(true, "Show Bearish BOS", group="BOS/CHoCH Settings")
showBullishCHoCH = input.bool(true, "Show Bullish CHoCH", group="BOS/CHoCH Settings")
showBearishCHoCH = input.bool(true, "Show Bearish CHoCH", group="BOS/CHoCH Settings")
chochCooldown = input.int(10, "CHoCH Cooldown (bars)", minval=5, maxval=50, group="BOS/CHoCH Settings")

// Supply/Demand Zones
extendZones = input.int(20, "Extend Zones (bars)", minval=5, maxval=100, group="Supply/Demand Settings")
zoneTransparency = input.int(85, "Zone Transparency", minval=50, maxval=95, group="Supply/Demand Settings")

// Support/Resistance
pivotLookback = input.int(10, "Pivot Lookback", minval=5, maxval=50, group="Support/Resistance")
showSRLevels = input.bool(true, "Show S/R Levels", group="Support/Resistance")

// Retracement Settings
retraceCandle1Size = input.float(50, "Retracement Candle 1 Min Body %", minval=20, maxval=100, group="Retracement Pattern")
retraceCandle2Size = input.float(50, "Retracement Candle 2 Min Body %", minval=20, maxval=100, group="Retracement Pattern")
cooldownBars = input.int(5, "2-CR Cooldown (bars)", minval=1, maxval=20, group="Retracement Pattern")

// ═════════════════════════════════════════════════════════════════════════════
// CALCULATIONS
// ═════════════════════════════════════════════════════════════════════════════

// Candle Properties
bodySize = math.abs(close - open)
candleRange = high - low
bodyPercent = (bodySize / candleRange) * 100
isStrongCandle = bodyPercent >= strongCandleThreshold
isBullish = close > open
isBearish = close < open

// Trend Detection
higherHighs = high > ta.highest(high[1], trendLookback)
lowerLows = low < ta.lowest(low[1], trendLookback)

// ═════════════════════════════════════════════════════════════════════════════
// MARKET STRUCTURE TRACKING
// ═════════════════════════════════════════════════════════════════════════════

var float lastSwingHigh = 0
var float lastSwingLow = 0
var float prevSwingHigh = 0
var float prevSwingLow = 0
var int lastSwingHighBar = 0
var int lastSwingLowBar = 0

if high > ta.highest(high[1], 5)
    prevSwingHigh := lastSwingHigh
    lastSwingHigh := high
    lastSwingHighBar := bar_index

if low < ta.lowest(low[1], 5)
    prevSwingLow := lastSwingLow
    lastSwingLow := low
    lastSwingLowBar := bar_index

var bool inBullishTrend = false
var bool inBearishTrend = false

if higherHighs and not inBullishTrend
    inBullishTrend := true
    inBearishTrend := false
if lowerLows and not inBearishTrend
    inBearishTrend := true
    inBullishTrend := false

// ═════════════════════════════════════════════════════════════════════════════
// BOS (BREAK OF STRUCTURE) - FILTERED
// ═════════════════════════════════════════════════════════════════════════════

var float lastBullHigh = 0
var float lastBullLow = 0
var int lastBullBar = 0
var int lastBOSTime = 0

var float lastBearLow = 0
var float lastBearHigh = 0
var int lastBearBar = 0

// Calculate minimum move threshold based on price
minMovePoints = close * bosMinMove / 100

// Bullish BOS: Break above high with cooldown and minimum move
bullishBOS_raw = high > ta.highest(high[1], bosLookback)
bullishBOS_valid = bullishBOS_raw and (high - lastBullHigh > minMovePoints or lastBullHigh == 0)
bullishBOS = bullishBOS_valid and (bar_index - lastBOSTime > bosCooldown)

if bullishBOS
    lastBullHigh := high
    lastBullLow := low
    lastBullBar := bar_index
    lastBOSTime := bar_index

// Bearish BOS: Break below low with cooldown and minimum move
bearishBOS_raw = low < ta.lowest(low[1], bosLookback)
bearishBOS_valid = bearishBOS_raw and (lastBearLow - low > minMovePoints or lastBearLow == 0)
bearishBOS = bearishBOS_valid and (bar_index - lastBOSTime > bosCooldown)

if bearishBOS
    lastBearLow := low
    lastBearHigh := high
    lastBearBar := bar_index
    lastBOSTime := bar_index

// ═════════════════════════════════════════════════════════════════════════════
// CHoCH (CHANGE OF CHARACTER) DETECTION
// ═════════════════════════════════════════════════════════════════════════════

var float lastCHoCHHigh = 0
var float lastCHoCHLow = 0
var int lastCHoCHBar = 0
var int lastCHoCHTime = 0

bullishCHoCH_raw = inBearishTrend[1] and high > lastSwingHigh[1] and bar_index > lastSwingHighBar[1] + 2
bullishCHoCH = bullishCHoCH_raw and (bar_index - lastCHoCHTime > chochCooldown)

bearishCHoCH_raw = inBullishTrend[1] and low < lastSwingLow[1] and bar_index > lastSwingLowBar[1] + 2
bearishCHoCH = bearishCHoCH_raw and (bar_index - lastCHoCHTime > chochCooldown)

if bullishCHoCH or bearishCHoCH
    lastCHoCHHigh := high
    lastCHoCHLow := low
    lastCHoCHBar := bar_index
    lastCHoCHTime := bar_index

if bullishCHoCH
    inBullishTrend := true
    inBearishTrend := false

if bearishCHoCH
    inBearishTrend := true
    inBullishTrend := false

// ═════════════════════════════════════════════════════════════════════════════
// 2-CANDLE RETRACEMENT PATTERN DETECTION
// ═════════════════════════════════════════════════════════════════════════════

var int last2CRBar = 0

candle1Bearish = isBearish and (bodyPercent >= retraceCandle1Size)
candle2Bearish = isBearish and (bodyPercent >= retraceCandle2Size)
candle1Bullish = isBullish and (bodyPercent >= retraceCandle1Size)
candle2Bullish = isBullish and (bodyPercent >= retraceCandle2Size)

bearishRetracement = inBullishTrend[2] and candle1Bearish[1] and candle2Bearish and (bar_index - last2CRBar > cooldownBars)
bullishRetracement = inBearishTrend[2] and candle1Bullish[1] and candle2Bullish and (bar_index - last2CRBar > cooldownBars)

if bullishRetracement or bearishRetracement
    last2CRBar := bar_index

// ═════════════════════════════════════════════════════════════════════════════
// SUPPLY & DEMAND ZONES
// ═════════════════════════════════════════════════════════════════════════════

var box demandZone = na
var label demandLabel = na

if bullishBOS or bullishCHoCH
    if not na(demandZone)
        box.delete(demandZone)
    if not na(demandLabel)
        label.delete(demandLabel)
    
    zoneTop = bullishCHoCH ? lastCHoCHHigh : lastBullHigh
    zoneBottom = bullishCHoCH ? lastCHoCHLow : lastBullLow
    
    demandZone := box.new(left=bar_index, top=zoneTop, right=bar_index+extendZones, bottom=zoneBottom, 
         bgcolor=color.new(color.green, zoneTransparency), border_color=color.green, border_width=1)
    
    zoneType = bullishCHoCH ? "CHoCH Demand" : "BOS Demand"
    demandLabel := label.new(bar_index, zoneTop, zoneType, color=color.new(color.green, 100), 
         textcolor=color.green, style=label.style_label_down, size=size.small)

var box supplyZone = na
var label supplyLabel = na

if bearishBOS or bearishCHoCH
    if not na(supplyZone)
        box.delete(supplyZone)
    if not na(supplyLabel)
        label.delete(supplyLabel)
    
    zoneTop = bearishCHoCH ? lastCHoCHHigh : lastBearHigh
    zoneBottom = bearishCHoCH ? lastCHoCHLow : lastBearLow
    
    supplyZone := box.new(left=bar_index, top=zoneTop, right=bar_index+extendZones, bottom=zoneBottom, 
         bgcolor=color.new(color.red, zoneTransparency), border_color=color.red, border_width=1)
    
    zoneType = bearishCHoCH ? "CHoCH Supply" : "BOS Supply"
    supplyLabel := label.new(bar_index, zoneBottom, zoneType, color=color.new(color.red, 100), 
         textcolor=color.red, style=label.style_label_up, size=size.small)

// Extend existing zones - FIXED LINE 209
if not na(demandZone)
    box.set_right(demandZone, bar_index + extendZones)
if not na(supplyZone)
    box.set_right(supplyZone, bar_index + extendZones)

// ═════════════════════════════════════════════════════════════════════════════
// SUPPORT & RESISTANCE LEVELS
// ═════════════════════════════════════════════════════════════════════════════

pivotHighVal = ta.pivothigh(high, pivotLookback, pivotLookback)
pivotLowVal = ta.pivotlow(low, pivotLookback, pivotLookback)

var line[] resistanceLines = array.new_line(0)
var line[] supportLines = array.new_line(0)

if not na(pivotHighVal) and showSRLevels
    newLine = line.new(bar_index - pivotLookback, pivotHighVal, bar_index + 20, pivotHighVal, 
         color=color.green, width=2, style=line.style_solid)
    array.push(resistanceLines, newLine)
    if array.size(resistanceLines) > 5
        line.delete(array.shift(resistanceLines))

if not na(pivotLowVal) and showSRLevels
    newLine = line.new(bar_index - pivotLookback, pivotLowVal, bar_index + 20, pivotLowVal, 
         color=color.red, width=2, style=line.style_solid)
    array.push(supportLines, newLine)
    if array.size(supportLines) > 5
        line.delete(array.shift(supportLines))

// ═════════════════════════════════════════════════════════════════════════════
// VISUALIZATION
// ═════════════════════════════════════════════════════════════════════════════

plotshape(isStrongCandle and isBullish, "Strong Bull", shape.triangleup, location.belowbar, color.green, size=size.small)
plotshape(isStrongCandle and isBearish, "Strong Bear", shape.triangledown, location.abovebar, color.red, size=size.small)

plotshape(bullishRetracement, "Bull 2-CR", shape.labelup, location.belowbar, color.lime, 
     text="2-CR", textcolor=color.white, size=size.normal, offset=-1)
plotshape(bearishRetracement, "Bear 2-CR", shape.labeldown, location.abovebar, color.orange, 
     text="2-CR", textcolor=color.white, size=size.normal, offset=-1)

plotshape(bullishBOS and showBullishBOS, "Bull BOS", shape.circle, location.belowbar, color.green, 
     text="BOS", textcolor=color.green, size=size.small)
plotshape(bearishBOS and showBearishBOS, "Bear BOS", shape.circle, location.abovebar, color.red, 
     text="BOS", textcolor=color.red, size=size.small)

plotshape(bullishCHoCH and showBullishCHoCH, "Bull CHoCH", shape.diamond, location.belowbar, color.blue, 
     text="CHoCH", textcolor=color.white, size=size.normal)
plotshape(bearishCHoCH and showBearishCHoCH, "Bear CHoCH", shape.diamond, location.abovebar, color.purple, 
     text="CHoCH", textcolor=color.white, size=size.normal)

// ═════════════════════════════════════════════════════════════════════════════
// INFORMATION PANEL
// ═════════════════════════════════════════════════════════════════════════════

var table infoPanel = table.new(position.top_right, 2, 5, bgcolor=color.new(color.black, 80), 
     frame_color=color.gray, frame_width=1, border_color=color.gray)

if barstate.islast
    table.cell(infoPanel, 0, 0, "Trend:", text_color=color.white, text_size=size.small)
    trendText = inBullishTrend ? "BULLISH" : inBearishTrend ? "BEARISH" : "NEUTRAL"
    trendColor = inBullishTrend ? color.green : inBearishTrend ? color.red : color.gray
    table.cell(infoPanel, 1, 0, trendText, text_color=trendColor, text_size=size.small)
    
    table.cell(infoPanel, 0, 1, "Last BOS:", text_color=color.white, text_size=size.small)
    lastBOSStr = bullishBOS ? "Bull" : bearishBOS ? "Bear" : "None"
    table.cell(infoPanel, 1, 1, lastBOSStr, text_color=color.yellow, text_size=size.small)
    
    table.cell(infoPanel, 0, 2, "Last CHoCH:", text_color=color.white, text_size=size.small)
    lastCHoCHStr = bullishCHoCH ? "Bull" : bearishCHoCH ? "Bear" : "None"
    table.cell(infoPanel, 1, 2, lastCHoCHStr, text_color=color.blue, text_size=size.small)
    
    table.cell(infoPanel, 0, 3, "BOS Cooldown:", text_color=color.white, text_size=size.small)
    bosCooldownStr = str.tostring(bar_index - lastBOSTime) + " bars"
    table.cell(infoPanel, 1, 3, bosCooldownStr, text_color=color.green, text_size=size.small)
    
    table.cell(infoPanel, 0, 4, "2-CR Cooldown:", text_color=color.white, text_size=size.small)
    cooldownStr = str.tostring(bar_index - last2CRBar) + " bars"
    table.cell(infoPanel, 1, 4, cooldownStr, text_color=color.orange, text_size=size.small)

// ═════════════════════════════════════════════════════════════════════════════
// ALERTS
// ═════════════════════════════════════════════════════════════════════════════

alertcondition(bullishBOS, "Bullish BOS", "Bullish Break of Structure Detected!")
alertcondition(bearishBOS, "Bearish BOS", "Bearish Break of Structure Detected!")
alertcondition(bullishCHoCH, "Bullish CHoCH", "Bullish Change of Character - Trend Reversal!")
alertcondition(bearishCHoCH, "Bearish CHoCH", "Bearish Change of Character - Trend Reversal!")
alertcondition(bullishRetracement, "Bullish Retracement", "2-Candle Bullish Retracement Pattern - Entry Signal!")
alertcondition(bearishRetracement, "Bearish Retracement", "2-Candle Bearish Retracement Pattern - Entry Signal!")

bgcolor(inBullishTrend ? color.new(color.green, 95) : inBearishTrend ? color.new(color.red, 95) : na, title="Trend Background")
