Code Header Block

This top section is like the EA's ID card. It names the file AUDUSD_Butterfly_Options_EA.mq5, adds copyright info, version number, and a description explaining what 
it does (trades butterfly options with specific risk rules). The #property strict makes the code follow strict MQL5 rules.

Include Libraries Block
#include <Trade\Trade.mqh>

These are like importing tools. Trade.mqh gives functions to place buy/sell orders, check positions, and manage trades automatically.

Input Parameters Block

These are settings users can change in MT5:

Account size: $50K (for calculating position sizes)

Risk per trade: 2% of account

Timeframe confirmation: H1 chart for trend signals

Option deltas: ATM=0.5 (middle strike), wings=0.16 (outer strikes)

Skew threshold: 0.05 (looks for favorable option pricing)

Risk rules: 2:1 profit target, exit if loses 10%

Drawdown limits: Pause at 10% loss, stop at 15%

Global Variables Block

gPeakEquity: Tracks highest account balance (for drawdown calc)

gPaused: Stops trading during big losses

gSymbol: AUDUSD pair

OptionLeg: Structure to track each option part (strike price, delta, etc.)

OnInit() - Startup Function
What it does when EA starts:

Sets AUDUSD as main symbol

Gets price precision (5 digits for AUDUSD)

Records starting account balance

Warns if actual balance ≠ $50K setting

Prints "EA ready" message

OnDeinit() - Shutdown Function
What happens when EA stops:
Simply prints why it stopped (manual close, crash, etc.)

OnTick() - Main Brain (Runs Every Price Update)
The heartbeat - executes continuously:

1. If paused → check if can resume
2. Check if drawdown limits hit → protect account
3. Count current trades → max 3 butterflies allowed
4. New 15min bar + H1 trend OK → check trading signal
5. Signal found → place butterfly trade
6. Manage all open positions (check profit/loss exits)

DetectContangoSkew() - Market Condition Check
Simplified volatility check:
Uses ATR (Average True Range) as "implied volatility proxy"
If ATR/price > 5%, assumes good butterfly setup
(This is simplified - real brokers provide actual option IV data)

IsVegaNeutral() - Risk Balance Check
Checks if trade is "vega neutral":
Wings (2 cheap options) should balance body (2 expensive options)
Math: |0.2×2 - 0.4×2| < 0.1 → balanced vol exposure

MultiTFConfirm() - Trend Confirmation
Makes sure trend aligns across timeframes:

15min close > 20-EMA AND

H1 20-EMA sloping up
= Bullish environment good for long butterflies

ExecuteButterfly() - Places The Actual Trade

Core trading logic:

1. Calculate risk: $50K × 2% = $1,000 max loss
2. Strikes: ITM=spot-50pips, ATM=spot, OTM=spot+50pips
3. Premiums: ITM=$30, ATM=$20×2, OTM=$30
4. Net cost: $30+$30-$40 = $20 debit
5. Lots: $1,000 ÷ $20 = 50 lots (scaled)
6. Place 4 orders: Buy ITM, Sell 2×ATM, Buy OTM




PlaceOptionOrder() - Order Function (Needs Broker Customization)
Placeholder for real option orders:
Currently uses spot AUDUSD as demo
MUST UPDATE with your broker's option symbols like:
AUDUSD.c-25.12.26-0.6500 (call expiring Dec 25th strike 0.6500)

ManagePositions() - Exit Logic
Monitors every open trade:

If profit ≥ 2×risk (200%) OR loss ≥ 10% → close immediately
Example: Risk $1K → close at +$2K profit OR -$100 loss

CountButterflies() - Position Counter
Loops through all trades, counts only this EA's positions (magic=123456)

CheckEquityProtector() - Account Safety Net
Critical risk control:

Current DD = (Peak Equity - Current) ÷ Peak Equity
If DD > 15% → EA SELFS DESTRUCTS
If DD > 10% → PAUSES TRADING
Tracks new peak equity when account grows

CheckResume() - Recovery Logic
Resumes when drawdown recovers 80% (10% → 8% loss)

IsNewBar() - Timing Control
Only trades on new 15-minute bars (prevents over-trading)

OnTester() - Backtest Optimization
For Strategy Tester genetic optimization:
Checks if backtest achieved:

25%+ profit over 3 years AND

Drawdown ≤15%
Prints success stats for parameter tuning

Production Notes
Compile: F7 in MetaEditor → .ex5 file created

Broker: Must support MT5 Options Board (CFI, FXCM, etc.)

Testing: Demo first, then $50K+ live account

VPS: Required for 24/7 execution

Customization: Update PlaceOptionOrder() for your broker's symbols

Overall Strategy: Waits for low-volatility "contango skew" + bullish trend → places cheap long butterfly (max $1K risk) → exits at 2:1 profit or 10% stop → repeats 
with strict drawdown controls. Institutional-grade risk management protects capital.





