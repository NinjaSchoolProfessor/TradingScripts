# Trading Scripts for Think or Swim
Scripts to improve insights on various trading platforms
MIT License - Free to use and modify

---

## Table of contents

1. [Volumatic VIDYA](#volumatic_vidya)
2. [Trend Magic](#trend-magic)
4. [SuperTrend](#supertrend)
5. [VWAP](#vwap)
6. [RSI](#rsi)
7. [StochRSI](#stochrsi)
8. [Opening Range Breakout](#opening-range-breakout)
9. [SuperTrend Stock Hacker Scanner](#super-trend-stock-hacker-scanner)
10. [Formatting](#formatting)

---

# Volumatic VIDYA

A thinkScript implementation of the Volumatic Variable Index Dynamic Average (VIDYA) indicator with volume pressure analysis for ThinkOrSwim (TOS).

## What is Volumatic VIDYA?

Volumatic VIDYA combines the Variable Index Dynamic Average (VIDYA) with volume pressure analysis to provide a comprehensive trend-following system. Unlike traditional moving averages that use fixed smoothing periods, VIDYA dynamically adjusts its responsiveness based on market momentum using the Chande Momentum Oscillator (CMO).

The indicator provides:

- **Adaptive trend line** that responds faster during strong momentum and slower during consolidation
- **ATR-based cloud** for volatility-adjusted support/resistance zones
- **Delta volume tracking** to measure buy/sell pressure within each trend
- **Buy/Sell signals** with visual bubbles and arrows at trend reversals
- **Summary labels** displaying real-time trend status and volume metrics
- **Customizable alerts** with selectable sounds for trend changes

## Visual Components

### VIDYA Line (Colored Trend Line)

The bright colored line running through price is the Variable Index Dynamic Average itself. This line dynamically adjusts its smoothing based on momentum (CMO) — during strong moves it reacts faster; during consolidation it smooths more.

| Line Color | Meaning |
|------------|---------|
| **Green** | Bullish trend active — price closed above upper band |
| **Red** | Bearish trend active — price closed below lower band |

The VIDYA line is more responsive than traditional moving averages while filtering out noise, making it ideal for trend identification.

### ATR Cloud (Shaded Volatility Envelope)

The colored cloud surrounding the VIDYA line represents the ATR-based volatility envelope:

| Band | Calculation |
|------|-------------|
| **Upper Band** | VIDYA + (ATR × Multiplier) |
| **Lower Band** | VIDYA - (ATR × Multiplier) |

**What the cloud tells you:**

| Price Action | What Happens | Interpretation |
|--------------|--------------|----------------|
| Price inside cloud | No signal generated | Trend continuation, noise filtered |
| Price closes above upper band | Line turns green, "Buy" signal | Bullish trend flip |
| Price closes below lower band | Line turns red, "Sell" signal | Bearish trend flip |

**Cloud characteristics:**

| Cloud Width | Meaning |
|-------------|---------|
| Wide cloud | High volatility environment |
| Narrow cloud | Low volatility / consolidation |

The cloud acts as a **noise filter**. Small pullbacks within the bands don't trigger trend changes — only decisive moves through the bands generate signals. When price is riding along the top of a green cloud, that's textbook bullish behavior: VIDYA trending up with price respecting the upper band as dynamic support.

### Buy/Sell Signals

| Signal | Appearance | Trigger |
|--------|------------|---------|
| **Buy** | Green arrow + "Buy" bubble below price | Price closed above upper band (trend flipped bullish) |
| **Sell** | Red arrow + "Sell" bubble above price | Price closed below lower band (trend flipped bearish) |

Signals only appear at trend changes, keeping the chart clean and actionable.

## Summary Labels

The indicator displays four summary labels at the top of the chart:

| Label | Color | Description |
|-------|-------|-------------|
| **VIDYA: Bullish** | Green | Uptrend active — price closed above upper band |
| **VIDYA: Bearish** | Red | Downtrend active — price closed below lower band |
| **Buy Vol: [value]** | Green | Cumulative volume on up bars (close > open) since last trend change |
| **Sell Vol: [value]** | Red | Cumulative volume on down bars (close < open) since last trend change |
| **Delta: [value]** | Green (positive) / Red (negative) | Net volume pressure (Buy Vol - Sell Vol) since trend started |

### Understanding Delta

**Delta = Buy Volume − Sell Volume** (during the current trend)

Delta resets to zero each time the trend flips.

| Delta | During Uptrend | During Downtrend |
|-------|----------------|------------------|
| **Large positive** | Confirms strength — buyers supporting the move | Warning — price falling but buyers active |
| **Small positive** | Weak buying — potential exhaustion | Mixed signals |
| **Large negative** | Warning — price rising but sellers active (divergence) | Confirms strength — sellers supporting the move |
| **Small negative** | Mixed signals | Weak selling — potential reversal |

**Example:** Buy Vol: 44.46K, Sell Vol: 40.28K, Delta: 4.18K (green) during a bullish trend confirms the uptrend has participation — buyers accumulated ~4.18K more volume than sellers.

## How It Works

### VIDYA Calculation

1. **Momentum Measurement**: Calculate price change over the momentum period
2. **CMO Calculation**: Separate positive and negative momentum sums to derive the absolute CMO value
3. **Dynamic Alpha**: `alpha = 2 / (length + 1)` adjusted by CMO percentage
4. **VIDYA Value**: `VIDYA = (alpha × |CMO| / 100 × price) + (1 - alpha × |CMO| / 100) × previous VIDYA`
5. **Smoothing**: Final value smoothed with a 15-period SMA

### Trend Detection

| Condition | Trend |
|-----------|-------|
| Price closes above upper band | Bullish |
| Price closes below lower band | Bearish |
| Price between bands | Previous trend continues |

### Volume Pressure (Delta Volume)

The indicator accumulates volume throughout each trend:

- **Up bars** (close > open) add to buy volume
- **Down bars** (close < open) add to sell volume
- **Delta** = Buy Volume - Sell Volume
- Delta resets on each trend change

## Quick Reference Tables

### Recommended Settings by Timeframe

| Timeframe | vidyaLength | vidyaMomentum | atrMult |
|-----------|-------------|---------------|---------|
| 1m | 12 | 28 | 2.8 |
| 5m | 10 | 20 | 2.4 |
| 15m | 10 | 18 | 2.0 |
| 1h | 10 | 16 | 1.7 |
| 4h | 8 | 14 | 1.6 |
| 1d | 8 | 14 | 1.5 |

### Settings by Trading Style

| Style | vidyaLength | vidyaMomentum | atrMult | Notes |
|-------|-------------|---------------|---------|-------|
| Aggressive | 8 | 16 | 2.0 | More signals, earlier entries |
| Balanced | 10 | 20 | 2.2 | Default, good all-around |
| Conservative | 12 | 25 | 2.8 | Fewer signals, higher quality |

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| timeFrame | 5m | Reference for recommended settings (1m, 5m, 15m, 1h, 4h, 1d) |
| vidyaLength | 10 | Base smoothing period for VIDYA calculation |
| vidyaMomentum | 20 | Lookback period for CMO calculation |
| atrMult | 2.4 | ATR multiplier for upper/lower bands (cloud width) |
| atrLength | 14 | Period for ATR calculation |
| showSummary | yes | Toggle summary labels on/off |
| bullishAlertSound | Ding | Sound for bullish alerts (Ding, Bell, Chimes, Ring, NoSound) |
| bearishAlertSound | Bell | Sound for bearish alerts (Ding, Bell, Chimes, Ring, NoSound) |

### Parameter Tuning

| Adjustment | Effect |
|------------|--------|
| **Higher vidyaLength** | Smoother line, slower to react |
| **Lower vidyaLength** | More responsive, more noise |
| **Higher vidyaMomentum** | More stable CMO, less adaptive |
| **Lower vidyaMomentum** | Faster adaptation, more sensitive |
| **Higher atrMult** | Wider cloud, fewer signals, higher quality |
| **Lower atrMult** | Tighter cloud, more signals, earlier entries |

**Note:** The `atrMult` is your **primary tuning knob**. Adjust this first, then fine-tune the other parameters if needed.

## Features

| Feature | Description |
|---------|-------------|
| **VIDYA Line** | Adaptive moving average with trend-based coloring (green/red) |
| **ATR Cloud** | Dynamic upper/lower bands with shaded fill showing volatility envelope |
| **Buy/Sell Arrows** | Visual arrows at trend reversals |
| **Buy/Sell Bubbles** | Labeled bubbles ("Buy"/"Sell") at trend changes only |
| **VIDYA Label** | Shows current trend direction (Bullish/Bearish) |
| **Buy Vol Label** | Cumulative volume on up bars since trend started |
| **Sell Vol Label** | Cumulative volume on down bars since trend started |
| **Delta Label** | Net volume pressure with color indicator |
| **Customizable Alerts** | Audio alerts with selectable sounds for trend changes |

## Installation

1. Open ThinkOrSwim
2. Go to **Charts** → **Studies** → **Edit Studies**
3. Click **Create** in the lower left
4. Name it "Volumatic_VIDYA"
5. Paste the code and click **OK**
6. Apply to chart

## Interpretation Guide

### Trend Analysis

| Signal | Meaning |
|--------|---------|
| Green VIDYA line | Bullish trend active |
| Red VIDYA line | Bearish trend active |
| Green "Buy" bubble + arrow | New bullish trend starting |
| Red "Sell" bubble + arrow | New bearish trend starting |
| Price riding top of green cloud | Strong bullish momentum, upper band acting as support |
| Price riding bottom of red cloud | Strong bearish momentum, lower band acting as resistance |

### Volume Analysis

| Delta Volume | Interpretation |
|--------------|----------------|
| Large positive | Strong buying pressure supporting uptrend |
| Small positive | Weak buying pressure, potential exhaustion |
| Large negative | Strong selling pressure supporting downtrend |
| Small negative | Weak selling pressure, potential reversal |

## VIDYA vs Traditional Moving Averages

| Aspect | SMA/EMA | VIDYA |
|--------|---------|-------|
| **Smoothing** | Fixed period | Adaptive to momentum |
| **Responsiveness** | Constant | Faster in trends, slower in ranges |
| **Whipsaws** | More frequent | Reduced due to dynamic adjustment |
| **Calculation** | Simple average | CMO-weighted smoothing |

## Use Cases

- **Trend Following**: Enter on buy/sell signals, ride trends with VIDYA as trailing reference
- **Volume Confirmation**: Validate trends by checking delta volume alignment
- **Momentum Assessment**: CMO-based adaptation reveals underlying momentum strength
- **Risk Management**: Use ATR cloud bands as dynamic stop-loss reference
- **Divergence Detection**: Watch for delta diverging from price direction

## Alert Sounds

Configure different sounds for bullish vs bearish alerts to quickly identify trend direction without looking at the chart:

| Sound Option | Description |
|--------------|-------------|
| Ding | Short, high-pitched tone (default for bullish) |
| Bell | Classic bell sound (default for bearish) |
| Chimes | Multi-tone chime |
| Ring | Ringing tone |
| NoSound | Silent (visual only) |

## Usage Notes

- Works on any timeframe and instrument
- Most effective in trending markets with clear directional moves
- Delta volume most meaningful on assets with reliable volume data (stocks, futures)
- The cloud width indicates current volatility — wider cloud = more volatile conditions
- Price riding the edge of the cloud confirms trend strength
- Consider combining with RSI or MACD for additional confirmation
- Customize alert sounds to distinguish between bullish and bearish signals
- Not financial advice — always backtest before live trading

## Credits

Inspired by the original [Volumatic VIDYA by BigBeluga](https://www.tradingview.com/script/llhVjhA5-Volumatic-Variable-Index-Dynamic-Average-BigBeluga/).

---

### Trend Magic
- Trend Magic (CCI-based) avoids noise and defines the big picture. (pair with Super Trend)
- Trend Magic is a trend-tracking line that uses the Commodity Channel Index (CCI) to determine the market’s directional state and the Average True Range (ATR) to set a dynamic band that acts as support or resistance. It turns bullish (blue) when the CCI value is above zero and bearish (red) when the CCI value is below zero.
```
# Trend Magic Indicator - Uses CCI to determine trend direction and ATR-based trailing stops
# Features:
# - Adaptive trailing stop that follows price based on CCI momentum
# - Blue line when CCI >= 0 (bullish), Red line when CCI < 0 (bearish)
# - Customizable CCI period, ATR multiplier, and ATR period
# - Multiple price source options (Close, Open, High, Low, HL2, HLC3, OHLC4)
# - Toggle between standard ATR (Wilder's) or SMA-based ATR calculation
# - Acts as dynamic support/resistance levels that adjust with volatility
#
# Updated via Anthropic Claude 20-Nov-2025 @ 6:45 PM EDT
#
# NinjaSchoolProfessor.com
# https://github.com/NinjaSchoolProfessor/TradingScripts

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

# ----- Label (green for bullish, red for bearish) -----
AddLabel(
    yes,
    if cciVal >= 0
    then "Magic Trend: Bullish"
    else "Magic Trend: Bearish",
    if cciVal >= 0
    then Color.GREEN
    else Color.RED
);
```

### SuperTrend
 - SuperTrend uses the Average True Range (ATR) to build adaptive support and resistance levels and signals an up or down trend by showing an “UP” or “DOWN” bubble when the market shifts into bullish or bearish conditions.
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)
 - Updated via Anthropic Claude 20-Nov-2025 @ 2:45 PM EDT

```
# SuperTrend Indicator for ThinkOrSwim
# 
# Traditional SuperTrend algorithm using ATR-based dynamic support/resistance bands.
# Tracks bullish (uptrend) and bearish (downtrend) conditions.
#
# FEATURES:
# - Invisible plot line (calculation-only, no visual clutter)
# - "UP" and "DOWN" bubbles at trend reversals
# - Color-coded label showing current trend state
# - Customizable audio alerts for trend changes
# - Selectable ATR calculation method (WILDERS, SIMPLE, HULL, etc.)
#
# KEY INPUTS:
# - length: ATR period (default 14)
# - multiplier: ATR multiplier for band width (default 1.0)
# - AvgType: Moving average type for ATR calculation
# - soundUp/soundDown: Alert sounds (use Sound.NoSound to disable)
#
# USAGE:
# Green = Bullish trend, Red = Bearish trend
# Trend changes trigger bubbles and optional audio alerts
#
# Updated via Anthropic Claude 20-Nov-2025 @ 3:14 PM EDT
#
# NinjaSchoolProfessor.com
# https://github.com/NinjaSchoolProfessor/TradingScripts

declare upper;

input length = 14;
input multiplier = 2.0;
input AvgType = AverageType.WILDERS;
input showBubbles = yes;
input showLabel = yes;
input soundUp = Sound.Chimes;
input soundDown = Sound.Bell;

# Calculate ATR
def ATR = MovingAverage(AvgType, TrueRange(high, close, low), length);

# Calculate basic bands
def upperBand = HL2 + (multiplier * ATR);
def lowerBand = HL2 - (multiplier * ATR);

# Calculate final bands with persistence logic
def finalUpperBand;
def finalLowerBand;

finalUpperBand = if upperBand < finalUpperBand[1] or close[1] > finalUpperBand[1] 
                 then upperBand 
                 else finalUpperBand[1];

finalLowerBand = if lowerBand > finalLowerBand[1] or close[1] < finalLowerBand[1] 
                 then lowerBand 
                 else finalLowerBand[1];

# Determine trend direction
def superTrend;
def trend;

trend = if close > finalUpperBand[1] then 1
        else if close < finalLowerBand[1] then -1
        else trend[1];

superTrend = if trend == 1 then finalLowerBand else finalUpperBand;

# Plot SuperTrend line (invisible)
plot ST = superTrend;
ST.SetDefaultColor(Color.CURRENT);
ST.Hide();

# Detect trend changes for bubbles
def trendChange = trend != trend[1];
def isBullish = trend == 1;

# Add bubbles at trend changes
AddChartBubble(showBubbles and trendChange and isBullish, low, "UP", Color.GREEN, no);
AddChartBubble(showBubbles and trendChange and !isBullish, high, "DOWN", Color.RED, yes);

# Add label
AddLabel(showLabel, 
         if isBullish then "SuperTrend: Up" else "SuperTrend: Down", 
         if isBullish then Color.GREEN else Color.RED);

# Audio alerts - plays once when trend changes
Alert(trendChange and isBullish, "SuperTrend Flip: Up", Alert.ONCE, soundUp);
Alert(trendChange and !isBullish, "SuperTrend Flip: Down", Alert.ONCE, soundDown);
```

### VWAP
- Standard VWAP with a label that turns green or red depending on the position of price action.
```
# Volume-Weighted Average Price (VWAP) Indicator
# Calculates the average price weighted by volume, resetting daily
# Features:
# - Color-coded based on price position relative to VWAP
# - Summary label showing current relationship
#
# VWAP is commonly used by institutional traders as a benchmark for execution quality
# Price above VWAP = Bullish, Price below VWAP = Bearish
#
# Created via Anthropic Claude 23-Nov-2025
# NinjaSchoolProfessor.com


input numDevDn = -0.0;
#hint numDevDn: "Default standard deviation is -2.0"
input numDevUp = 0.0;
#hint numDevUp: "Default standard deviation is 2.0"
input timeFrame = {default DAY, WEEK, MONTH};

def cap = getAggregationPeriod();
def errorInAggregation =
    timeFrame == timeFrame.DAY and cap >= AggregationPeriod.WEEK or
    timeFrame == timeFrame.WEEK and cap >= AggregationPeriod.MONTH;
assert(!errorInAggregation, "timeFrame should be not less than current chart aggregation period");

def yyyyMmDd = getYyyyMmDd();
def periodIndx;
switch (timeFrame) {
case DAY:
    periodIndx = yyyyMmDd;
case WEEK:
    periodIndx = Floor((daysFromDate(first(yyyyMmDd)) + getDayOfWeek(first(yyyyMmDd))) / 7);
case MONTH:
    periodIndx = roundDown(yyyyMmDd / 100, 0);
}

def isPeriodRolled = compoundValue(1, periodIndx != periodIndx[1], yes);

def volumeSum;
def volumeVwapSum;
def volumeVwap2Sum;

if (isPeriodRolled) {
    volumeSum = volume;
    volumeVwapSum = volume * vwap;
    volumeVwap2Sum = volume * Sqr(vwap);
} else {
    volumeSum = compoundValue(1, volumeSum[1] + volume, volume);
    volumeVwapSum = compoundValue(1, volumeVwapSum[1] + volume * vwap, volume * vwap);
    volumeVwap2Sum = compoundValue(1, volumeVwap2Sum[1] + volume * Sqr(vwap), volume * Sqr(vwap));
}

def price = volumeVwapSum / volumeSum;
def deviation = Sqrt(Max(volumeVwap2Sum / volumeSum - Sqr(price), 0));

plot VWAP = price;
plot UpperBand = price + numDevUp * deviation;
plot LowerBand = price + numDevDn * deviation;

VWAP.setDefaultColor(getColor(0));
UpperBand.setDefaultColor(getColor(2));
LowerBand.setDefaultColor(getColor(4));

# VWAP Cross Label
def priceAboveVWAP = close > VWAP;
def priceBelowVWAP = close < VWAP;

AddLabel(yes, 
    if priceAboveVWAP then "VWAP: Bullish" 
    else if priceBelowVWAP then "VWAP: Bearish" 
    else "VWAP: Neutral", 
    if priceAboveVWAP then Color.GREEN 
    else if priceBelowVWAP then Color.RED 
    else Color.GRAY);
```

### RSI
- Standard relative strength index (RSI) that includes a label that turns green/red as StochRSI moves above/below 50%. Then turns cyan for over bought, and orage for over sold.
```
declare lower;
input length = 14;
input over_Bought = 70;
#hint over_Bought: Default signal for over bought is 70.
input over_Sold = 30;
#hint over_Sold: Default signal for over sold is 30.
input price = close;
input averageType = AverageType.WILDERS;
input showBreakoutSignals = no;

def NetChgAvg = MovingAverage(averageType, price - price[1], length);
def TotChgAvg = MovingAverage(averageType, AbsValue(price - price[1]), length);
def ChgRatio = if TotChgAvg != 0 then NetChgAvg / TotChgAvg else 0;

plot RSI = 50 * (ChgRatio + 1);
plot OverSold = over_Sold;
plot OverBought = over_Bought;
plot UpSignal = if RSI crosses above OverSold then OverSold else Double.NaN;
plot DownSignal = if RSI crosses below OverBought then OverBought else Double.NaN;

UpSignal.SetHiding(!showBreakoutSignals);
DownSignal.SetHiding(!showBreakoutSignals);

RSI.DefineColor("OverBought", GetColor(5));
RSI.DefineColor("Normal", GetColor(7));
RSI.DefineColor("OverSold", GetColor(1));
RSI.AssignValueColor(if RSI > over_Bought then RSI.color("OverBought") else if RSI < over_Sold then RSI.color("OverSold") else RSI.color("Normal"));

OverSold.SetDefaultColor(GetColor(8));
OverBought.SetDefaultColor(GetColor(8));
UpSignal.SetDefaultColor(Color.UPTICK);
UpSignal.SetPaintingStrategy(PaintingStrategy.ARROW_UP);
DownSignal.SetDefaultColor(Color.DOWNTICK);
DownSignal.SetPaintingStrategy(PaintingStrategy.ARROW_DOWN);

AddLabel(yes, 
    if RSI > over_Bought then "RSI: Over bought" 
    else if RSI < over_Sold then "RSI: Over sold" 
    else if RSI > 50 then "RSI: > 50" 
    else "RSI: < 50", 
    if RSI > over_Bought then Color.CYAN 
    else if RSI < over_Sold then Color.ORANGE 
    else if RSI > 50 then Color.GREEN 
    else Color.RED);
```

### StochRSI
- Standard stochastic relative strength index (StochRSI) that includes a label that turns green/red as StochRSI moves above/below 50%. Then turns cyan for over bought, and orage for over sold.

```
declare lower;

input RSI_length = 14;
input over_bought = 80;
input over_sold = 20;
input RSI_average_type = AverageType.WILDERS;
input RSI_price = close;
input KPeriod = 14;
input DPeriod = 3;
input slowing_period = 1;
input averageType = AverageType.SIMPLE;
input showBreakoutSignals = {default "No", "On FullK", "On FullD", "On FullK & FullD"};

def RSI = RSI(price = RSI_price, length = RSI_length, averageType = RSI_average_type);

plot FullK = StochasticFull(over_bought, over_sold, KPeriod, DPeriod, RSI, RSI, RSI, slowing_period, averageType).FullK;
plot FullD = StochasticFull(over_bought, over_sold, KPeriod, DPeriod, RSI, RSI, RSI, slowing_period, averageType).FullD;
plot OverBought = over_bought;
plot OverSold = over_sold;

def upK = FullK crosses above OverSold;
def upD = FullD crosses above OverSold;
def downK = FullK crosses below OverBought;
def downD = FullD crosses below OverBought;

plot UpSignal;
plot DownSignal;
switch (showBreakoutSignals) {
case "No":
    UpSignal = Double.NaN;
    DownSignal = Double.NaN;
case "On FullK":
    UpSignal = if upK then OverSold else Double.NaN;
    DownSignal = if downK then OverBought else Double.NaN;
case "On FullD":
    UpSignal = if upD then OverSold else Double.NaN;
    DownSignal = if downD then OverBought else Double.NaN;
case "On FullK & FullD":
    UpSignal = if upK or upD then OverSold else Double.NaN;
    DownSignal = if downK or downD then OverBought else Double.NaN;
}

UpSignal.setHiding(showBreakoutSignals == showBreakoutSignals."No");
DownSignal.setHiding(showBreakoutSignals == showBreakoutSignals."No");

FullK.SetDefaultColor(GetColor(5));
FullD.SetDefaultColor(GetColor(0));
OverBought.SetDefaultColor(GetColor(1));
OverSold.SetDefaultColor(GetColor(1));
UpSignal.SetDefaultColor(Color.UPTICK);
UpSignal.SetPaintingStrategy(PaintingStrategy.ARROW_UP);
DownSignal.SetDefaultColor(Color.DOWNTICK);
DownSignal.SetPaintingStrategy(PaintingStrategy.ARROW_DOWN);

AddLabel(yes, 
    if FullK > over_Bought then "StochRSI: Over bought" 
    else if FullK < over_Sold then "StochRSI: Over sold" 
    else if FullK > 50 then "StochRSI: > 50" 
    else "StochRSI: < 50", 
    if FullK > over_Bought then Color.CYAN 
    else if FullK < over_Sold then Color.ORANGE 
    else if FullK > 50 then Color.GREEN 
    else Color.RED);
```
### Opening Range Breakout

## This study is currently a work in progress -- DO NOT USE YET --

- This script identifies and tracks Opening Range Breakouts (ORB) for intraday trading. It measures the high and low during a user-defined opening period, typically the first 15 minutes, then visualizes that range on the chart. The script flags breakouts when price moves and closes above the opening high for a potential bullish setup or below the opening low for a potential bearish setup. When a breakout occurs, it also calculates ATR-based profit targets to help with trade planning. The previous day’s close is displayed for added context, and a real-time status label shows the current ORB direction.
- Original Source: [https://usethinkscript.com/threads/opening-range-breakout-indicator-for-thinkorswim.16/](https://usethinkscript.com/threads/opening-range-breakout-indicator-for-thinkorswim.16/)
```
# OPENING RANGE BREAKOUT (ORB) INDICATOR
# This indicator identifies and tracks Opening Range Breakouts for intraday trading.
# 
# WHAT IT DOES:
# - Tracks the high/low range during a user-defined opening period (default: first 15 minutes)
# - Creates visual clouds showing the opening range
# - Detects when price breaks above (bullish) or below (bearish) the opening range
# - Calculates ATR-based profit targets when breakouts occur
# - Displays previous day's close for reference
# - Provides real-time status label showing current ORB direction
#
# BREAKOUT TYPES:
# - "On Close": Breakout confirmed when candle closes outside range (conservative)
# - "On Wick Touch": Breakout triggered when wick touches range (aggressive)
#
# BEST USED ON: 5-minute charts or lower for precise entry timing
#
# Updated via Anthropic Claude 20-Nov-2025 @ 6:20 PM
#
# NinjaSchoolProfessor.com
# https://github.com/NinjaSchoolProfessor/TradingScripts
# ================================================================================

declare Hide_On_Daily;
declare Once_per_bar;

# ================================================================================
# USER INPUTS
# ================================================================================

# --- Opening Range Settings ---
input OR_Begin = 0930.0; 
#hint OR_Begin: Start time for Opening Range period in ET (e.g., 0930 = 9:30 AM). This is when the range calculation begins.

input OR_Duration = {default "15 min", "5 min", "30 min", "1 hour"}; 
#hint OR_Duration: How long to track the opening range. 15 min is most common for day trading.

input Show_Today_Only = {"No", default "Yes"}; 
#hint Show_Today_Only: "Yes" = Only show ORB for current trading day. "No" = Show ORB for all historical days.

# --- Breakout Detection ---
input ORB_Breakout_Type = {default "On Close", "On Wick Touch"}; 
#hint ORB_Breakout_Type: "On Close" = Breakout confirmed when candle closes outside range (safer). "On Wick Touch" = Breakout triggers when wick touches range (faster but more false signals).

# --- Visual Elements ---
input Cloud_On = yes; 
#hint Cloud_On: Show/hide the colored cloud visualization of the opening range.

input Show_ORB_Signal_Bubbles = no; 
#hint Show_ORB_Signal_Bubbles: Show "ORB Bullish" or "ORB Bearish" bubbles when breakout occurs.

input ORB_Bubble_Location = {"Inside ORB Bubble", default "Outside ORB Bubble"}; 
#hint ORB_Bubble_Location: "Outside" = Bubbles appear away from range. "Inside" = Bubbles appear toward the range.

input Show_ORB_Label = yes; 
#hint Show_ORB_Label: Show the ORB status label in upper left (N/A, Bullish, or Bearish).

# --- Target Settings ---
input ATR_Length = 4; 
#hint ATR_Length: Number of bars to calculate Average True Range for profit targets. Default 4 works well for 5-min charts.

input ATR_Target_Multiplier = 2.0; 
#hint ATR_Target_Multiplier: Multiplier for ATR to calculate target distances. 2.0 = targets are 2x ATR away from breakout.

input Target_1_Visible = yes; 
#hint Target_1_Visible: Show/hide the first profit target line.

input Target_2_Visible = no; 
#hint Target_2_Visible: Show/hide the second profit target line.

input Target_3_Visible = no; 
#hint Target_3_Visible: Show/hide the third profit target line.

input Target_4_Visible = no; 
#hint Target_4_Visible: Show/hide the fourth profit target line.

input Target_5_Visible = no; 
#hint Target_5_Visible: Show/hide the fifth profit target line.

input Show_Target_Bubbles = no; 
#hint Show_Target_Bubbles: Show/hide text labels at the right edge for each target level.

# --- Alert Settings ---
input Alert_On = yes; 
#hint Alert_On: Enable/disable audio alerts when breakouts occur.

# ================================================================================
# CORE VARIABLES & CALCULATIONS
# ================================================================================

def high_price = high;
def low_price = low;
def close_price = close;
def bar_number = barNumber();
def show_today = Show_Today_Only;
def UseWickTouch = ORB_Breakout_Type == ORB_Breakout_Type."On Wick Touch";
def na = double.nan;

# Convert duration selection to seconds
def DurationSeconds = if OR_Duration == OR_Duration."5 min" then 300
                      else if OR_Duration == OR_Duration."15 min" then 900
                      else if OR_Duration == OR_Duration."30 min" then 1800
                      else 3600; # 1 hour

# Determine if we're looking at today
def today = if show_today == 0
            or getDay() == getLastDay() and secondsFromTime(OR_Begin) >= 0
            then 1
            else 0;
			
# ================================================================================
# OPENING RANGE DETECTION
# ================================================================================

# Check if we're currently within the Opening Range period
def OpenRangeActive = if secondsFromTime(OR_Begin) >= 0 and
                         secondsFromTime(OR_Begin) < DurationSeconds
                      then 1
                      else 0;

# Track the highest high during Opening Range
def OpenRangeHigh = if OpenRangeHigh[1] == 0
                or OpenRangeActive[1] == 0 and OpenRangeActive == 1
              then high_price
              else if OpenRangeActive and high_price > OpenRangeHigh[1]
              then high_price
              else OpenRangeHigh[1];

# Track the lowest low during Opening Range
def OpenRangeLow = if OpenRangeLow[1] == 0
              or OpenRangeActive[1] == 0 and OpenRangeActive == 1
             then low_price
             else if OpenRangeActive and low_price < OpenRangeLow[1]
             then low_price
             else OpenRangeLow[1];

def OpenRangeWidth = OpenRangeHigh - OpenRangeLow;

# Mark the bar where Opening Range ended
def OpenRangeEndBar = if !OpenRangeActive and OpenRangeActive[1]
               then barNumber()
               else OpenRangeEndBar[1];

# Values for plotting (only show after OR completes)
def OpenRangeHighValue = if OpenRangeActive or today < 1
                         then na
                         else OpenRangeHigh;

def OpenRangeLowValue = if OpenRangeActive or today < 1
                        then na
                        else OpenRangeLow;
# ================================================================================
# CLOUD VISUALIZATION
# ================================================================================

# Calculate extended cloud values for visualization
def OpenRangeHighCloud = if bar_number >= highestAll(OpenRangeEndBar)
                then HighestAll(if isNaN(close_price[-1])
                                then OpenRangeHighValue[1]
                                else double.nan)
                else double.nan;

def OpenRangeLowCloud = if bar_number >= highestAll(OpenRangeEndBar)
                then HighestAll(if isNaN(close_price[-1])
                                then OpenRangeLowValue[1]
                                else double.nan)
                else double.nan;

# Calculate midpoint to split the cloud into two colors
def OpenRangeMidpoint = if bar_number >= highestAll(OpenRangeEndBar)
                        then HighestAll(if isNaN(close_price[-1])
                                       then (OpenRangeHighValue[1] + OpenRangeLowValue[1]) / 2
                                       else double.nan)
                        else double.nan;

# Green cloud above midpoint (upper half of range)
addCloud(if Cloud_On == yes then OpenRangeHighCloud else double.nan, 
         OpenRangeMidpoint, 
         createColor(180, 255, 200), 
         createColor(180, 255, 200));

# Red cloud below midpoint (lower half of range)
addCloud(if Cloud_On == yes then OpenRangeMidpoint else double.nan, 
         OpenRangeLowCloud, 
         createColor(255, 200, 190), 
         createColor(255, 200, 190));
		 
# ================================================================================
# BREAKOUT DETECTION & TARGET CALCULATIONS
# ================================================================================

# --- Breakout Conditions (based on user selection) ---
def BullishBreakoutCondition = if UseWickTouch 
                               then high_price > OpenRangeHigh and high_price[1] <= OpenRangeHigh
                               else close_price crosses above OpenRangeHigh;

def BearishBreakoutCondition = if UseWickTouch
                               then low_price < OpenRangeLow and low_price[1] >= OpenRangeLow
                               else close_price crosses below OpenRangeLow;

# --- ATR Calculation ---
def AverageTrueRange = if OpenRangeActive
                       then Round((Average(TrueRange(high_price, close_price, low_price), ATR_Length)) / TickSize(), 0) * TickSize()
                       else AverageTrueRange[1];

# --- Upper Targets (Bullish Breakout) ---
# First target is set when price first breaks above OR High
def FirstUpperTarget = if today and high_price > OpenRangeHigh and high_price[1] <= OpenRangeHigh
                       then Round((OpenRangeHigh + (AverageTrueRange * ATR_Target_Multiplier)) / TickSize(), 0) * TickSize()
                       else if today then FirstUpperTarget[1]
                       else 0;

# Subsequent targets trigger as price reaches each level
def SecondUpperTarget = if today and close_price crosses above FirstUpperTarget
                        then Round((FirstUpperTarget + (AverageTrueRange * ATR_Target_Multiplier)) / TickSize(), 0) * TickSize()
                        else if today then SecondUpperTarget[1]
                        else 0;

def ThirdUpperTarget = if today and close_price crosses above SecondUpperTarget
                       then Round((SecondUpperTarget + (AverageTrueRange * ATR_Target_Multiplier)) / TickSize(), 0) * TickSize()
                       else if today then ThirdUpperTarget[1]
                       else 0;

def FourthUpperTarget = if today and close_price crosses above ThirdUpperTarget
                        then Round((ThirdUpperTarget + (AverageTrueRange * ATR_Target_Multiplier)) / TickSize(), 0) * TickSize()
                        else if today then FourthUpperTarget[1]
                        else 0;

def FifthUpperTarget = if today and close_price crosses above FourthUpperTarget
                       then Round((FourthUpperTarget + (AverageTrueRange * ATR_Target_Multiplier)) / TickSize(), 0) * TickSize()
                       else if today then FifthUpperTarget[1]
                       else 0;

# --- Lower Targets (Bearish Breakout) ---
# First target is set when price first breaks below OR Low
def FirstLowerTarget = if today and low_price < OpenRangeLow and low_price[1] >= OpenRangeLow
                       then Round((OpenRangeLow - (ATR_Target_Multiplier * AverageTrueRange)) / TickSize(), 0) * TickSize()
                       else if today then FirstLowerTarget[1]
                       else 0;

# Subsequent targets trigger as price reaches each level
def SecondLowerTarget = if today and close_price crosses below FirstLowerTarget
                        then Round((FirstLowerTarget - (ATR_Target_Multiplier * AverageTrueRange)) / TickSize(), 0) * TickSize()
                        else if today then SecondLowerTarget[1]
                        else 0;

def ThirdLowerTarget = if today and close_price crosses below SecondLowerTarget
                       then Round((SecondLowerTarget - (ATR_Target_Multiplier * AverageTrueRange)) / TickSize(), 0) * TickSize()
                       else if today then ThirdLowerTarget[1]
                       else 0;

def FourthLowerTarget = if today and close_price crosses ThirdLowerTarget
                        then Round((ThirdLowerTarget - (ATR_Target_Multiplier * AverageTrueRange)) / TickSize(), 0) * TickSize()
                        else if today then FourthLowerTarget[1]
                        else 0;

def FifthLowerTarget = if today and close_price crosses FourthLowerTarget
                       then Round((FourthLowerTarget - (ATR_Target_Multiplier * AverageTrueRange)) / TickSize(), 0) * TickSize()
                       else if today then FifthLowerTarget[1]
                       else 0;

# ================================================================================
# SIGNAL BUBBLES & STATUS LABEL
# ================================================================================

def BubbleLocation = isNaN(close_price[-1]);
def BubbleOutside = ORB_Bubble_Location == ORB_Bubble_Location."Outside ORB Bubble";

# --- Bullish Breakout Bubble ---
def CrossUpBar = if BullishBreakoutCondition then bar_number else double.nan;

plot BullishSignalLine = if bar_number >= OpenRangeEndBar and !OpenRangeActive and today
                         then HighestAll(OpenRangeHighCloud - 2)
                         else double.nan;
BullishSignalLine.SetStyle(Curve.Long_Dash);
BullishSignalLine.SetDefaultColor(Color.PINK);
BullishSignalLine.HideTitle();
BullishSignalLine.Hide();

AddChartBubble(Show_ORB_Signal_Bubbles and bar_number == HighestAll(CrossUpBar), 
               OpenRangeHighCloud, 
               "ORB Bullish", 
               Color.PINK, 
               BubbleOutside);

# --- Bearish Breakout Bubble ---
def CrossDownBar = if BearishBreakoutCondition then bar_number else double.nan;

plot BearishSignalLine = if bar_number >= OpenRangeEndBar and !OpenRangeActive and close_price < OpenRangeLow
                         then HighestAll(OpenRangeLowCloud + 2)
                         else double.nan;
BearishSignalLine.SetStyle(Curve.Long_Dash);
BearishSignalLine.SetDefaultColor(Color.PINK);
BearishSignalLine.HideTitle();
BearishSignalLine.Hide();

AddChartBubble(Show_ORB_Signal_Bubbles and bar_number == HighestAll(CrossDownBar), 
               HighestAll(OpenRangeLowCloud), 
               "ORB Bearish", 
               Color.PINK, 
               !BubbleOutside);

# --- ORB Status Label (tracks most recent breakout) ---
# 0 = No breakout yet, 1 = Bullish, -1 = Bearish
def ORB_Status = if !today then 0
                 else if BullishBreakoutCondition then 1
                 else if BearishBreakoutCondition then -1
                 else ORB_Status[1];

AddLabel(Show_ORB_Label, 
         if ORB_Status == 1 then "ORB: Bullish" 
         else if ORB_Status == -1 then "ORB: Bearish" 
         else "ORB: N/A",
         if ORB_Status == 1 then Color.GREEN
         else if ORB_Status == -1 then Color.RED
         else Color.GRAY);

# ================================================================================
# PLOT TARGET LINES
# ================================================================================

# --- Upper Target Lines (Blue Dashed) ---
plot UpperTarget1 = if bar_number >= highestAll(OpenRangeEndBar) and Target_1_Visible and FirstUpperTarget > 0
                    then FirstUpperTarget else double.nan;
UpperTarget1.SetStyle(Curve.LONG_DASH);
UpperTarget1.SetLineWeight(1);
UpperTarget1.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, UpperTarget1, "Target 1", Color.BLUE, if close_price > UpperTarget1 then no else yes);

plot UpperTarget2 = if bar_number >= highestAll(OpenRangeEndBar) and Target_2_Visible and SecondUpperTarget > 0
                    then SecondUpperTarget else double.nan;
UpperTarget2.SetStyle(Curve.LONG_DASH);
UpperTarget2.SetLineWeight(1);
UpperTarget2.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, UpperTarget2, "Target 2", Color.BLUE, if close_price > UpperTarget2 then no else yes);

plot UpperTarget3 = if bar_number >= highestAll(OpenRangeEndBar) and Target_3_Visible and ThirdUpperTarget > 0
                    then ThirdUpperTarget else double.nan;
UpperTarget3.SetStyle(Curve.LONG_DASH);
UpperTarget3.SetLineWeight(1);
UpperTarget3.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, UpperTarget3, "Target 3", Color.BLUE, if close_price > UpperTarget3 then no else yes);

plot UpperTarget4 = if bar_number >= highestAll(OpenRangeEndBar) and Target_4_Visible and FourthUpperTarget > 0
                    then FourthUpperTarget else double.nan;
UpperTarget4.SetStyle(Curve.LONG_DASH);
UpperTarget4.SetLineWeight(1);
UpperTarget4.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, UpperTarget4, "Target 4", Color.BLUE, if close_price > UpperTarget4 then no else yes);

plot UpperTarget5 = if bar_number >= highestAll(OpenRangeEndBar) and Target_5_Visible and FifthUpperTarget > 0
                    then FifthUpperTarget else double.nan;
UpperTarget5.SetStyle(Curve.LONG_DASH);
UpperTarget5.SetLineWeight(1);
UpperTarget5.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, UpperTarget5, "Target 5", Color.BLUE, if close_price > UpperTarget5 then no else yes);

# --- Lower Target Lines (Blue Dashed) ---
plot LowerTarget1 = if bar_number >= highestAll(OpenRangeEndBar) and Target_1_Visible
                    then highestAll(if isNaN(close_price[-1]) then FirstLowerTarget else double.nan)
                    else double.nan;
LowerTarget1.SetStyle(Curve.LONG_DASH);
LowerTarget1.SetLineWeight(1);
LowerTarget1.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, FirstLowerTarget, "Target 1", Color.BLUE, if close_price < LowerTarget1 then yes else no);

plot LowerTarget2 = if bar_number >= highestAll(OpenRangeEndBar) and Target_2_Visible
                    then highestAll(if isNaN(close_price[-1]) then SecondLowerTarget else double.nan)
                    else double.nan;
LowerTarget2.SetStyle(Curve.LONG_DASH);
LowerTarget2.SetLineWeight(1);
LowerTarget2.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, SecondLowerTarget, "Target 2", Color.BLUE, if close_price < LowerTarget2 then yes else no);

plot LowerTarget3 = if bar_number >= highestAll(OpenRangeEndBar) and Target_3_Visible
                    then highestAll(if isNaN(close_price[-1]) then ThirdLowerTarget else double.nan)
                    else double.nan;
LowerTarget3.SetStyle(Curve.LONG_DASH);
LowerTarget3.SetLineWeight(1);
LowerTarget3.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, ThirdLowerTarget, "Target 3", Color.BLUE, if close_price < LowerTarget3 then yes else no);

plot LowerTarget4 = if bar_number >= highestAll(OpenRangeEndBar) and Target_4_Visible
                    then highestAll(if isNaN(close_price[-1]) then FourthLowerTarget else double.nan)
                    else double.nan;
LowerTarget4.SetStyle(Curve.LONG_DASH);
LowerTarget4.SetLineWeight(1);
LowerTarget4.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, FourthLowerTarget, "Target 4", Color.BLUE, if close_price < LowerTarget4 then yes else no);

plot LowerTarget5 = if bar_number >= highestAll(OpenRangeEndBar) and Target_5_Visible
                    then highestAll(if isNaN(close_price[-1]) then FifthLowerTarget else double.nan)
                    else double.nan;
LowerTarget5.SetStyle(Curve.LONG_DASH);
LowerTarget5.SetLineWeight(1);
LowerTarget5.SetDefaultColor(Color.BLUE);
AddChartBubble(Show_Target_Bubbles and BubbleLocation, FifthLowerTarget, "Target 5", Color.BLUE, if close_price < LowerTarget5 then yes else no);

# ================================================================================
# PREVIOUS CLOSE REFERENCE LINE
# ================================================================================

def PreviousClosePrice = if secondsTillTime(1600) == 0 and secondsFromTime(1600) == 0
                         then close_price[1]
                         else PreviousClosePrice[1];

plot PreviousClose = if today and PreviousClosePrice != 0
                     then PreviousClosePrice
                     else Double.NaN;
PreviousClose.SetPaintingStrategy(PaintingStrategy.TRIANGLES);
PreviousClose.SetDefaultColor(Color.YELLOW);
PreviousClose.HideBubble();
PreviousClose.HideTitle();

AddChartBubble(SecondsTillTime(0930) == 0, PreviousClose, "Previous Close", Color.GRAY, yes);

# ================================================================================
# ALERTS
# ================================================================================

alert(BullishBreakoutCondition, "ORB Bullish Breakout", Alert.Bar, Sound.Bell);
alert(BearishBreakoutCondition, "ORB Bearish Breakout", Alert.Bar, Sound.Ring);

# End of Opening Range Breakout Indicator
```

### SuperTrend Stock Hacker Scanner
 - Originial Source: [https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/](https://usethinkscript.com/threads/supertrend-indicator-by-mobius-for-thinkorswim.7/)

### Formatting
- [https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN](https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN)
