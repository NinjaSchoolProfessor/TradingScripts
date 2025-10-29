# TradingScripts
Scripts to improve insights on various trading platforms

## ThinkOrSwim ThinkScripts

### SuperTrend Indicator
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)
 - Modified using ChatGPT on 29-Oct-2025

```
# SuperTrend (ATR 21, Multiplier 1)
# Mobius-style structure, cleaned up for TOS
input AtrMult = 1.0;          # <- multiplier = 1
input nATR    = 21;           # <- length = 21
input PaintBars = yes;

# ATR using Wilder averaging, typical for SuperTrend
def ATR = MovingAverage(AverageType.WILDERS, TrueRange(high, close, low), nATR);

# Basic bands
def UpperBand = HL2 + AtrMult * ATR;
def LowerBand = HL2 - AtrMult * ATR;

# Final bands that "carry" until price invalidates them
def PrevFinalUpper = CompoundValue(1, if IsNaN(PrevFinalUpper[1]) then UpperBand else PrevFinalUpper[1], UpperBand);
def PrevFinalLower = CompoundValue(1, if IsNaN(PrevFinalLower[1]) then LowerBand else PrevFinalLower[1], LowerBand);

def FinalUpper = CompoundValue(
    1,
    if UpperBand < PrevFinalUpper or close[1] > PrevFinalUpper
    then UpperBand
    else PrevFinalUpper,
    UpperBand
);

def FinalLower = CompoundValue(
    1,
    if LowerBand > PrevFinalLower or close[1] < PrevFinalLower
    then LowerBand
    else PrevFinalLower,
    LowerBand
);

# Trend direction and SuperTrend line
def Trend = CompoundValue(
    1,
    if IsNaN(Trend[1]) then 1
    else if Trend[1] == -1 and close > FinalUpper[1] then 1
    else if Trend[1] ==  1 and close < FinalLower[1] then -1
    else Trend[1],
    1
);

def ST = if Trend == 1 then FinalLower else FinalUpper;

plot SuperTrend = ST;
SuperTrend.AssignValueColor(if Trend == 1 then Color.GREEN else Color.RED);
SuperTrend.SetLineWeight(2);

AssignPriceColor(
    if PaintBars then
        if Trend == 1 then Color.GREEN else Color.RED
    else Color.CURRENT
);

# Optional bubbles on flips
AddChartBubble(Trend crosses above 0, low, "Flip Up", Color.GREEN, no);
AddChartBubble(Trend crosses below 0, high, "Flip Down", Color.RED, yes);
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
