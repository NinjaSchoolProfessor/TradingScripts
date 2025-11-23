# TradingScripts
Scripts to improve insights on various trading platforms

---
## Table of contents

1. [Trend Magic](#trend-magic)
2. [SuperTrend](#supertrend)
3. [VWAP](#vwap)
4. [RSI](#rsi)
5. [Opening Range Breakout](#opening-range-breakout)
6. [SuperTrend Stock Hacker Scanner](#super-trend-stock-hacker-scanner)
7. [Formatting](#formatting)

---

## ThinkOrSwim - ToS - ThinkScripts

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
# - Customizable price source (Typical Price, Close, HL2, HLC3, OHLC4)
# - Optional standard deviation bands (1σ, 2σ, 3σ)
# - Daily reset at market open
# - Color-coded based on price position relative to VWAP
# - Summary label showing current relationship
#
# VWAP is commonly used by institutional traders as a benchmark for execution quality
# Price above VWAP = Bullish, Price below VWAP = Bearish
#
# Created via Anthropic Claude 23-Nov-2025
# NinjaSchoolProfessor.com

declare upper;
declare once_per_bar;

input priceSource = {default TYPICAL, CLOSE, HL2, HLC3, OHLC4};
input timeFrame = AggregationPeriod.DAY;
input showStdDevBands = yes;
input numStdDev1 = 1.0;
input numStdDev2 = 2.0;
input numStdDev3 = 3.0;
input showLabel = yes;

# Define price based on user selection
def price;
switch (priceSource) {
    case CLOSE:
        price = close;
    case HL2:
        price = (high + low) / 2;
    case HLC3:
        price = (high + low + close) / 3;
    case OHLC4:
        price = (open + high + low + close) / 4;
    default:  # TYPICAL
        price = (high + low + close) / 3;
}

# Check if we're at the start of a new time period (9:30 AM ET market open)
def isNewPeriod = if SecondsFromTime(0930) == 0 
                  then yes 
                  else if GetDay() != GetDay()[1] and SecondsFromTime(0930) >= 0
                  then yes
                  else no;

# Cumulative calculations that reset each period
def cumVolume = if isNewPeriod then volume else cumVolume[1] + volume;
def cumVolPrice = if isNewPeriod then volume * price else cumVolPrice[1] + (volume * price);

# Calculate VWAP
def vwapValue = cumVolPrice / cumVolume;

# Calculate standard deviation for bands
def priceDev = price - vwapValue;
def priceDevSq = priceDev * priceDev;
def cumPriceDevSq = if isNewPeriod then priceDevSq * volume else cumPriceDevSq[1] + (priceDevSq * volume);
def variance = cumPriceDevSq / cumVolume;
def stdDev = Sqrt(variance);

# Plot VWAP line
plot VWAP = vwapValue;
VWAP.SetDefaultColor(Color.CYAN);
VWAP.SetLineWeight(2);
VWAP.SetStyle(Curve.FIRM);

# Upper Standard Deviation Bands
plot UpperBand1 = if showStdDevBands then vwapValue + (numStdDev1 * stdDev) else Double.NaN;
UpperBand1.SetDefaultColor(Color.LIGHT_GREEN);
UpperBand1.SetStyle(Curve.SHORT_DASH);

plot UpperBand2 = if showStdDevBands then vwapValue + (numStdDev2 * stdDev) else Double.NaN;
UpperBand2.SetDefaultColor(Color.GREEN);
UpperBand2.SetStyle(Curve.SHORT_DASH);

plot UpperBand3 = if showStdDevBands then vwapValue + (numStdDev3 * stdDev) else Double.NaN;
UpperBand3.SetDefaultColor(Color.DARK_GREEN);
UpperBand3.SetStyle(Curve.SHORT_DASH);

# Lower Standard Deviation Bands
plot LowerBand1 = if showStdDevBands then vwapValue - (numStdDev1 * stdDev) else Double.NaN;
LowerBand1.SetDefaultColor(Color.LIGHT_RED);
LowerBand1.SetStyle(Curve.SHORT_DASH);

plot LowerBand2 = if showStdDevBands then vwapValue - (numStdDev2 * stdDev) else Double.NaN;
LowerBand2.SetDefaultColor(Color.RED);
LowerBand2.SetStyle(Curve.SHORT_DASH);

plot LowerBand3 = if showStdDevBands then vwapValue - (numStdDev3 * stdDev) else Double.NaN;
LowerBand3.SetDefaultColor(Color.DARK_RED);
LowerBand3.SetStyle(Curve.SHORT_DASH);

# Optional cloud between upper and lower 1σ bands
AddCloud(UpperBand1, LowerBand1, Color.LIGHT_GRAY, Color.LIGHT_GRAY);

# Determine current position relative to VWAP
def priceAboveVWAP = close > vwapValue;

# Add label showing bullish/bearish status
AddLabel(
    showLabel,
    if priceAboveVWAP
    then "VWAP: Bullish"
    else "VWAP: Bearish",
    if priceAboveVWAP
    then Color.GREEN
    else Color.RED);

# Optional: Add bubbles at day open showing VWAP anchor point
def showBubble = isNewPeriod and !IsNaN(close);
AddChartBubble(showBubble, vwapValue, "VWAP Start", Color.CYAN, yes);
```

### RSI
- Standard relative strength index (RSI) that includes a label that turns green/red as RSI moves above a given threshold.
```
# RSI Label Indicator

# Inputs
input length = 14;
input upperThreshold = 70;
input lowerThreshold = 30;
input showLabel = yes;

# Calculate RSI
def rsi = RSI(length = length);

# Determine RSI condition
def rsiOverbought = rsi > upperThreshold;
def rsiOversold = rsi < lowerThreshold;

# Add Label
AddLabel(
    showLabel,
    if rsiOverbought
    then "RSI: Overbought"
    else if rsiOversold
    then "RSI: Oversold"
    else "RSI: Neutral",
    if rsiOverbought or rsiOversold
    then Color.ORANGE
    else Color.GRAY
);
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

### Formatting
- [https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN](https://toslc.thinkorswim.com/center/reference/thinkScript/Constants/Color/Color-GREEN)
