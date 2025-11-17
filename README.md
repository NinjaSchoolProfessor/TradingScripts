# TradingScripts
Scripts to improve insights on various trading platforms

---
## Table of contents

1. [Trend Magic](#trend-magic)
2. [Super Trend](#super-trend)
3. [Super Trend Stock Hacker Scanner](#super-trend-stock-hacker-scanner)
4. [Opening Range Breakout](#opening-range-breakout)
5. [Formatting](#formatting)

---

## ThinkOrSwim - ToS - ThinkScripts

### Trend Magic
- Trend Magic (CCI-based) avoids noise and defines the big picture. (pair with Super Trend)
- Trend Magic is a trend-tracking line that uses the Commodity Channel Index (CCI) to determine the market’s directional state and the Average True Range (ATR) to set a dynamic band that acts as support or resistance. It turns bullish (blue) when the CCI value is above zero and bearish (red) when the CCI value is below zero.
```
declare upper;

input CCIPeriod   = 20;
input ATRPeriod   = 5;
input ATRMult     = 1.0;
input PriceSource = {default Close, Open, High, Low};

# Select price source
def src =
if PriceSource == PriceSource.Close then close
else if PriceSource == PriceSource.Open then open
else if PriceSource == PriceSource.High then high
else low;

# ----- Manual CCI Calculation -----
def tp   = src;
def sma  = Average(tp, CCIPeriod);
def mad  = Average(AbsValue(tp - sma), CCIPeriod);
def cciVal = (tp - sma) / (0.015 * mad);

# ----- ATR -----
def tr  = TrueRange(high, close, low);
def atr = Average(tr, ATRPeriod);

# ----- Bands -----
def upT   = low  - (atr * ATRMult);
def downT = high + (atr * ATRMult);

# ----- Trend Magic Recursive Line -----
def MagicTrend =
CompoundValue(
    1,
    if cciVal >= 0 then
        if upT < MagicTrend[1]
        then MagicTrend[1]
        else upT
    else
        if downT > MagicTrend[1]
        then MagicTrend[1]
        else downT,
    src
);

# ----- Plot -----
plot TM = MagicTrend;
TM.SetLineWeight(3);
TM.AssignValueColor(
    if cciVal >= 0 then Color.BLUE else Color.RED
);
```

### Super Trend
 - Super Trend (price-based) reacts to breakouts. (pair with Trend Magic)
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)
 - Modified using ChatGPT on 29-Oct-2025

```# Super Trend (No Bar or Line Coloring)
# Based on Mobius version with no chart coloring, added label (top left of chart) and audio alerts to indicate change in trend direction
# Added audio alerts for Up (Chimes) and Down (Bell)
# Alternate sounds (Ring, Alarm) commented out for convenience

input AtrMult = 1.0;
input nATR = 21;
input AvgType = AverageType.WILDERS;   # WILDERS is classic; try SIMPLE or HULL if you want it faster
input showBubbles = yes;
input showLabel = yes;

def ATR = MovingAverage(AvgType, TrueRange(high, close, low), nATR);
def UP = HL2 + (AtrMult * ATR);
def DN = HL2 - (AtrMult * ATR);

rec ST = if IsNaN(ST[1]) then HL2 
         else if close < ST[1] then UP 
         else DN;

plot SuperTrend = ST;
SuperTrend.SetDefaultColor(Color.GRAY);
SuperTrend.SetLineWeight(2);

# --- Trend Detection ---
def isUp = close > ST;
def isDown = close < ST;
def crossUp = close crosses above ST;
def crossDown = close crosses below ST;

# --- Optional Bubbles ---
AddChartBubble(showBubbles and crossUp, low, "Up", Color.GREEN, no);
AddChartBubble(showBubbles and crossDown, high, "Down", Color.RED, yes);

# --- Optional Label ---
AddLabel(showLabel, 
         "SuperTrend: " + if isUp then "Up" else "Down", 
         if isUp then Color.GREEN else Color.RED);

# --- Audio Alerts ---
# Plays once per new trend change
Alert(crossUp, "SuperTrend Flip: Up", Alert.ONCE, Sound.Chimes);
Alert(crossDown, "SuperTrend Flip: Down", Alert.ONCE, Sound.Bell);

# --- Alternate sounds you can use instead ---
# Alert(crossUp, "SuperTrend Flip: Up", Alert.ONCE, Sound.Ring);
# Alert(crossDown, "SuperTrend Flip: Down", Alert.ONCE, Sound.Alarm);

```

### Super Trend Stock Hacker Scanner
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)
 - Modified using ChatGPT on 29-Oct-2025

```
# SuperTrend Scan (ATR 21, Multiplier 1)
# Based on Mobius logic, adapted for scanning
input AtrMult = 1.0;     # multiplier = 1
input nATR    = 21;      # ATR length = 21

def h = high;
def l = low;
def c = close;

# Wilder ATR
def ATR = MovingAverage(AverageType.WILDERS, TrueRange(h, c, l), nATR);

# Raw bands
def UpperBand = HL2 + AtrMult * ATR;
def LowerBand = HL2 - AtrMult * ATR;

# Final bands that carry until invalidated
def FUp = CompoundValue(
    1,
    if UpperBand < FUp[1] or c[1] > FUp[1] then UpperBand else FUp[1],
    UpperBand
);

def FLo = CompoundValue(
    1,
    if LowerBand > FLo[1] or c[1] < FLo[1] then LowerBand else FLo[1],
    LowerBand
);

# Trend state
def Trend = CompoundValue(
    1,
    if IsNaN(Trend[1]) then 1
    else if Trend[1] == -1 and c > FUp[1] then 1
    else if Trend[1] ==  1 and c < FLo[1] then -1
    else Trend[1],
    1
);

# SuperTrend line
def ST = if Trend == 1 then FLo else FUp;

# Round for scan stability
def ST_R = Round(ST / tickSize(), 0) * tickSize();

# Choose one direction for the scan
# plot SuperTrendFlipUp   = if c crosses above ST_R then 1 else 0;
plot SuperTrendFlipDown = if c crosses below ST_R then 1 else 0;
```

### Opening Range Breakout
- Opening Range Breakout (ORB) is a trading strategy that identifies the high and low during the early part of a trading session. A breakout occurs when the price moves and closes above the initial high or below the initial low.
- Original Source: [https://usethinkscript.com/threads/opening-range-breakout-indicator-for-thinkorswim.16/](https://usethinkscript.com/threads/opening-range-breakout-indicator-for-thinkorswim.16/)
```
# TI Opening Range Breakout with Bull/Bear Zones + Label + Alerts + Scan
# Time inputs are in Eastern Time

#############################
# USER INPUTS
#############################

input openingRangeStartTime = 0930;  #hint openingRangeStartTime: Start time of the opening range (e.g., 0930 for 9:30am ET).
input openingRangeEndTime   = 0945;  #hint openingRangeEndTime: End time of the opening range (e.g., 0945 for 9:45am ET).

input tradeEntryStartTime   = 1000;  #hint tradeEntryStartTime: Start time of the trade entry window where ORB breakouts are valid.
input tradeEntryEndTime     = 1100;  #hint tradeEntryEndTime: End time of the trade entry window where ORB breakouts are valid.

input sessionEndTime        = 1600;  #hint sessionEndTime: End of the regular session (e.g., 1600 for 4:00pm ET).

input entryType = { default Wick_Touch, Close };
#hint entryType: Choose how to define a breakout: Wick_Touch = high/low pierces the OR level; Close = candle close breaks the OR level.

# Display toggles
input showOpeningRangeBox   = yes;   #hint showOpeningRangeBox: Show the opening range high/low lines during the opening range.
input showOpeningRangeCloud = no;    #hint showOpeningRangeCloud: Show a gray shaded cloud for the opening range box between the OR high and OR low.
input showTradeEntryLines   = yes;   #hint showTradeEntryLines: Show dashed horizontal lines extending OR high/low through the trade entry window.
input showBullBearZones     = yes;   #hint showBullBearZones: Master toggle to show or hide both Bull (green) and Bear (pink) OR zones.
input showBullZone          = yes;   #hint showBullZone: Show the Bull Zone (midRange to OR high), typically the preferred long zone.
input showBearZone          = yes;   #hint showBearZone: Show the Bear Zone (OR low to midRange), typically the preferred short zone.
input showProfitTargetProjections = yes;
#hint showProfitTargetProjections: Show profit-target projection lines. When enabled, you get two upside targets after a bullish breakout (HalfUp = midpoint between OR high and OR high + range, FullUp = OR range added above OR high ~1R) and two downside targets after a bearish breakout (HalfDown = midpoint between OR low and OR low - range, FullDown = OR range subtracted below OR low ~1R). These appear only if a breakout occurred, projections are enabled, and the breakout direction matches the projection.
input showORBArrows         = yes;   #hint showORBArrows: Show ORB breakout markers (implemented as chart bubbles labeled "ORB Bull" / "ORB Bear").

# Bull/Bear zone style: clouds only (behind price) or lines + clouds
input bullBearZoneStyle = { default Clouds_Only, Lines_and_Clouds };
#hint bullBearZoneStyle: Choose how to display Bull/Bear zones: Clouds_Only = shaded zones behind price only; Lines_and_Clouds = add boundary lines along with the shaded zones.

# ORB bubble placement
input orbBubblePosition = { default Standard, Above, Below };
#hint orbBubblePosition: Choose where to place ORB bubbles. Standard = bull bubble below and bear bubble above the candle; Above = both bubbles above the candle; Below = both bubbles below the candle.

# Label behavior: Off, reset daily, or persistent until next signal
input labelMode = { default Daily_Reset, Persistent_Until_Next_Signal, Off };
#hint labelMode: Controls the ORB label behavior. Daily_Reset = first ORB direction of the day and reset next day; Persistent_Until_Next_Signal = label stays until the next opposite ORB signal; Off = no ORB label.

# Alerts
input enableORBAlerts = yes;         #hint enableORBAlerts: Enable or disable audible alerts when a bullish or bearish ORB breakout first triggers.
input orbAlertSound   = Sound.Ding;  #hint orbAlertSound: Sound to play when an ORB breakout alert fires.

#############################
# RANGE DEFINITIONS
#############################

# Opening range active between start and end
def openingRange =
    SecondsTillTime(openingRangeStartTime) <= 0 and
    SecondsTillTime(openingRangeEndTime)   >= 0;

# Trade entry window (time during which we look for the breakout)
def tradeEntryRange =
    SecondsTillTime(tradeEntryStartTime) <= 0 and
    SecondsTillTime(tradeEntryEndTime)   >= 0;

# Session active after opening range is locked in until sessionEndTime
def sessionActive =
    SecondsFromTime(openingRangeEndTime) >= 0 and
    SecondsTillTime(sessionEndTime) >= 0;

# Day boundaries for daily reset logic
def day    = GetDay();
def newDay = day <> day[1];

#############################
# OPENING RANGE HIGH / LOW
#############################

def openingRangeHigh =
    if SecondsTillTime(openingRangeStartTime) == 0 then
        high
    else if openingRange and high > openingRangeHigh[1] then
        high
    else
        openingRangeHigh[1];

def openingRangeLow =
    if SecondsTillTime(openingRangeStartTime) == 0 then
        low
    else if openingRange and low < openingRangeLow[1] then
        low
    else
        openingRangeLow[1];

#############################
# PLOTS – OPENING RANGE BOX (LINES + OPTIONAL CLOUD)
#############################

plot HighCloud =
    if showOpeningRangeBox and openingRange then openingRangeHigh else Double.NaN;
plot LowCloud =
    if showOpeningRangeBox and openingRange then openingRangeLow  else Double.NaN;

HighCloud.SetDefaultColor(Color.GRAY);
LowCloud.SetDefaultColor(Color.GRAY);

# Separate series for the optional gray OR cloud
def ORCloudLowSeries =
    if showOpeningRangeCloud and showOpeningRangeBox and openingRange
    then openingRangeLow
    else Double.NaN;

def ORCloudHighSeries =
    if showOpeningRangeCloud and showOpeningRangeBox and openingRange
    then openingRangeHigh
    else Double.NaN;

AddCloud(ORCloudLowSeries, ORCloudHighSeries, Color.GRAY, Color.GRAY);

#############################
# PLOTS – TRADE ENTRY EXTENSIONS
#############################

plot TradeEntryHighExtension =
    if showTradeEntryLines and tradeEntryRange then openingRangeHigh else Double.NaN;
plot TradeEntryLowExtension  =
    if showTradeEntryLines and tradeEntryRange then openingRangeLow  else Double.NaN;

TradeEntryHighExtension.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
TradeEntryLowExtension.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);

TradeEntryHighExtension.SetStyle(Curve.SHORT_DASH);
TradeEntryLowExtension.SetStyle(Curve.SHORT_DASH);

TradeEntryHighExtension.SetDefaultColor(Color.GRAY);
TradeEntryLowExtension.SetDefaultColor(Color.GRAY);

#############################
# ENTRY TYPE SWITCH
#############################

def bullEntryCondition;
def bearEntryCondition;

switch (entryType) {
case Wick_Touch:
    bullEntryCondition =
        tradeEntryRange and
        high > openingRangeHigh and
        high[1] <= openingRangeHigh[1];

    bearEntryCondition =
        tradeEntryRange and
        low < openingRangeLow and
        low[1] >= openingRangeLow[1];

case Close:
    bullEntryCondition =
        tradeEntryRange and
        close > openingRangeHigh and
        close[1] <= openingRangeHigh[1];

    bearEntryCondition =
        tradeEntryRange and
        close < openingRangeLow and
        close[1] >= openingRangeLow[1];
}

#############################
# PERSISTENT ORB FLAGS (INTRA-DAY)
#############################

def bullishORB =
    if bullEntryCondition then
        1
    else if !tradeEntryRange then
        0
    else
        bullishORB[1];

def bearishORB =
    if bearEntryCondition then
        1
    else if !tradeEntryRange then
        0
    else
        bearishORB[1];

#############################
# RANGE STATS AND MID RANGE
#############################

def range      = openingRangeHigh - openingRangeLow;
def halfRange  = range / 2;
def midRange   = openingRangeLow + halfRange;

plot MidRangePlot =
    if tradeEntryRange then midRange else Double.NaN;

MidRangePlot.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
MidRangePlot.SetStyle(Curve.SHORT_DASH);
MidRangePlot.SetDefaultColor(Color.LIGHT_GRAY);

#############################
# BULL / BEAR ZONES (CLOUDS + OPTIONAL LINES)
#############################

def bullZoneOn = showBullBearZones and showBullZone and sessionActive;
def bearZoneOn = showBullBearZones and showBearZone and sessionActive;

# Series used for clouds (always)
def BullZoneBottomSeries =
    if bullZoneOn then midRange else Double.NaN;
def BullZoneTopSeries =
    if bullZoneOn then openingRangeHigh else Double.NaN;

def BearZoneBottomSeries =
    if bearZoneOn then openingRangeLow else Double.NaN;
def BearZoneTopSeries =
    if bearZoneOn then midRange else Double.NaN;

# Optional line plots (only visible when Lines_and_Clouds selected)
plot BullZoneBottomLine;
plot BullZoneTopLine;
plot BearZoneBottomLine;
plot BearZoneTopLine;

switch (bullBearZoneStyle) {
case Clouds_Only:
    BullZoneBottomLine = Double.NaN;
    BullZoneTopLine    = Double.NaN;
    BearZoneBottomLine = Double.NaN;
    BearZoneTopLine    = Double.NaN;
case Lines_and_Clouds:
    BullZoneBottomLine = BullZoneBottomSeries;
    BullZoneTopLine    = BullZoneTopSeries;
    BearZoneBottomLine = BearZoneBottomSeries;
    BearZoneTopLine    = BearZoneTopSeries;
}

BullZoneBottomLine.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
BullZoneTopLine.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
BearZoneBottomLine.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
BearZoneTopLine.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);

BullZoneBottomLine.SetDefaultColor(Color.LIGHT_GREEN);
BullZoneTopLine.SetDefaultColor(Color.LIGHT_GREEN);
BearZoneBottomLine.SetDefaultColor(Color.PINK);
BearZoneTopLine.SetDefaultColor(Color.PINK);

BullZoneBottomLine.HideTitle();
BullZoneTopLine.HideTitle();
BearZoneBottomLine.HideTitle();
BearZoneTopLine.HideTitle();

# Clouds (always drawn behind price)
AddCloud(BullZoneBottomSeries, BullZoneTopSeries, Color.LIGHT_GREEN, Color.LIGHT_GREEN);
AddCloud(BearZoneBottomSeries, BearZoneTopSeries, Color.PINK, Color.PINK);

#############################
# ENTRY MARKERS (RAW SIGNALS)
#############################

def bullSignalCond = bullishORB and !bullishORB[1];
def bearSignalCond = bearishORB and !bearishORB[1];

#############################
# ORB BUBBLES ("ORB Bull" / "ORB Bear")
#############################

def bullBubbleAbove =
    orbBubblePosition == orbBubblePosition.Above;

def bearBubbleAbove =
    orbBubblePosition == orbBubblePosition.Above
    or orbBubblePosition == orbBubblePosition.Standard;

AddChartBubble(
    showORBArrows and bullSignalCond,
    low,
    "ORB Bull",
    Color.WHITE,
    bullBubbleAbove
);

AddChartBubble(
    showORBArrows and bearSignalCond,
    high,
    "ORB Bear",
    Color.WHITE,
    bearBubbleAbove
);

#############################
# PROFIT-TARGET PROJECTIONS (EXTENDED TO END OF DAY)
#############################

def halfwayUpsideProjection = openingRangeHigh + halfRange;
def fullUpsideProjection    = openingRangeHigh + range;

def halfwayDownsideProjection = openingRangeLow - halfRange;
def fullDownsideProjection    = openingRangeLow - range;

# Latch targets at breakout and hold for the rest of the day, reset at newDay
def bullHalfTP =
    if newDay then Double.NaN
    else if bullSignalCond then halfwayUpsideProjection
    else bullHalfTP[1];

def bullFullTP =
    if newDay then Double.NaN
    else if bullSignalCond then fullUpsideProjection
    else bullFullTP[1];

def bearHalfTP =
    if newDay then Double.NaN
    else if bearSignalCond then halfwayDownsideProjection
    else bearHalfTP[1];

def bearFullTP =
    if newDay then Double.NaN
    else if bearSignalCond then fullDownsideProjection
    else bearFullTP[1];

# Plots: extend across the entire right side of the chart once set
plot HalfUp =
    if showProfitTargetProjections and !IsNaN(bullHalfTP)
    then bullHalfTP
    else Double.NaN;

plot FullUp =
    if showProfitTargetProjections and !IsNaN(bullFullTP)
    then bullFullTP
    else Double.NaN;

HalfUp.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
FullUp.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);

HalfUp.SetDefaultColor(Color.LIGHT_GREEN);
FullUp.SetDefaultColor(Color.GREEN);
FullUp.SetLineWeight(2);

plot HalfDown =
    if showProfitTargetProjections and !IsNaN(bearHalfTP)
    then bearHalfTP
    else Double.NaN;

plot FullDown =
    if showProfitTargetProjections and !IsNaN(bearFullTP)
    then bearFullTP
    else Double.NaN;

HalfDown.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);
FullDown.SetPaintingStrategy(PaintingStrategy.HORIZONTAL);

HalfDown.SetDefaultColor(Color.LIGHT_RED);
FullDown.SetDefaultColor(Color.RED);
FullDown.SetLineWeight(2);

#############################
# DAILY / PERSISTENT ORB STATE (FOR LABEL)
#############################

# Daily-reset state
def orbStateDaily =
    if newDay then 0
    else if bullSignalCond then 1
    else if bearSignalCond then -1
    else orbStateDaily[1];

# Persistent-until-next-signal state
def orbStatePersistent =
    if bullSignalCond then 1
    else if bearSignalCond then -1
    else orbStatePersistent[1];

def useLabel = labelMode != labelMode.Off;

def orbState =
    if labelMode == labelMode.Daily_Reset then orbStateDaily
    else if labelMode == labelMode.Persistent_Until_Next_Signal then orbStatePersistent
    else 0;

#############################
# ORB LABEL
#############################

AddLabel(
    useLabel and orbState != 0,
    if orbState == 1
    then "ORB bullish breakout"
    else "ORB bearish breakout",
    if orbState == 1
    then Color.GREEN
    else Color.RED
);

#############################
# ORB ALERTS
#############################

Alert(
    enableORBAlerts and bullSignalCond,
    "ORB bullish breakout",
    Alert.BAR,
    orbAlertSound
);

Alert(
    enableORBAlerts and bearSignalCond,
    "ORB bearish breakout",
    Alert.BAR,
    orbAlertSound
);

#############################
# SCAN-FRIENDLY PLOTS
#############################
# Use these in a Scan: plot == true

plot BullORBScan = bullSignalCond;
BullORBScan.HideTitle();
BullORBScan.Hide();

plot BearORBScan = bearSignalCond;
BearORBScan.HideTitle();
BearORBScan.Hide();
```

### Formatting
- [https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN](https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN)
