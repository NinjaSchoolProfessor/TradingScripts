# TradingScripts
Scripts to improve insights on various trading platforms

## ThinkOrSwim ThinkScripts

### SuperTrend Indicator
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)
 - Modified using ChatGPT on 29-Oct-2025

```# SuperTrend (No Bar or Line Coloring)
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

### SuperTrend Stock Hacker Scanner
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
