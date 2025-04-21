# 01_Scalp_Strategy.md - 20× – BTC‑Alpha + Dynamic ETH/SOL Hedge

# Scope

Automated high‑frequency scalping bot on Binance Futures (Isolated 20×), driven by LuxAlgo indicators on TradingView, executed via FastAPI. Targets 62–68 % win‑rate, PF ≥ 1.6, ≤ 2.5 R day‐draw‑down, with BTC‑alpha core and dynamic ETH/SOL hedging.

# 01_Scalp_Strategy.md – BTC‑alpha + dynamic Hedge ETH/SOL

# 1 Indicator Configuration
1.1 LuxAlgo – Smart‐Money‐Concepts v1.4 (SMC)

    HTF: 15 min

    LTF: 3 min

    Volumetric OB: ON, Strength ≥ 30, Dominance ≥ 60 %

    BOS/CHoCH: ON (internal + swing)

    Premium/Discount zones: ON

    Mitigation logic: CLOSE

1.2 LuxAlgo – Price Action Concepts™

    BOS/CHoCH confirmations (framework alignment)

    Order‑Block precision: uses fair‑value gaps & liquidity pools to fine‑tune OB edges

    Liquidity grabs: ON (high‑precision structure filter)

    Equal highs/lows: Identify actionable liquidity clusters

1.3 LuxAlgo – Oscillator Matrix v6.1

    Money-Flow: length 34, auto thresholds, overflow ON

    Hyperwave: length 14, signal SMA 3

    Divergences: realtime, sensitivity 2

1.4 LuxAlgo – Signals & Overlays v5.3

    Mode: Confirmation, Autopilot ON

    Smart-Trail: Trail + Direction filter

    Classifier: Ratings ≥ 2

1.5 LuxAlgo – AI SuperTrend Clustering v2.0

    ATR: length 10

    Factor range: 1–3, step 0.5

    Memory: last 10 trades

    Cluster: BEST

# 2 Trading Logic (LONG side)
2.1 Context Filter

    15 m internal structure bullish (SMC + Price Action BOS)

    Price in discount half of 15 m range (SMC Premium/Discount + PAC fair‑value zones)

    Money‑Flow above threshold OR recent overflow ended

    Funding‑rate < 0.06 %

2.2 Trigger Sequence

    Price taps Volumetric Demand OB (≥ 60 % buy) and PAC‑defined OB edge

    Hyperwave crosses up below 50‑line inside OB

    No active overflow

    Buy Confirmation + bullish Smart‑Trail

2.3 Entry & Stop‑Loss

    Entry: Post‑only limit at mid‑OB (maker) → IOC market fallback after 700 ms

    SL: OB‑low – 0.6–0.75× ATR14 (use 0.6× when OB height < 1.2× ATR14)

2.4 Trade Management

    TP‑1 (30 %): +1 R, then move SL→BE

    TP‑2 (30 %): +1.5 R

    Trail (40 %): AI_SuperTrend dynamic trailing

    Hard exit: Opposite CHoCH (Price Action structure flip) or 30 min timeout

2.5 Dynamic ETH/SOL Hedge

    Calculate rolling 1 h β for ETH & SOL vs BTC

    On BTC LONG: open ETH & SOL SHORT sized = BTC_size×β×hedge_factor (0.5–0.7)

    Hedge SL = OB_high/low ± 0.9×ATR (no TP; close with BTC)

# 3 Trading Logic (SHORT side)

Mirror LONG logic with bearish bias:

    Context: 15 m bearish structure + price in premium half

    Trigger: Supply OB ≥ 60 % sell + Hyperwave down‑cross >50

    Entry & SL: Limit at mid‑OB (maker), fallback IOC; SL = OB‑high + 0.6–0.75×ATR

    Management: TP‑1 @ 1 R, TP‑2 @ 1.5 R, trail remainder via AI_SuperTrend, exit on CHoCH or timeout

    Hedge: On BTC SHORT, open ETH/SOL LONG same β×factor sizing

# 4 Alert‑Scripting (TradingView)

alert_message = "{HTF_BULL} AND {OB_DEMAND_HIT} AND {HW_CROSS_UP} AND {NO_OVERFLOW} AND {BUY_CONF} AND {TRAIL_BULL}"
alert(alert_message, alert.freq_all)

Webhook JSON (paste in TradingView alert):

{
  "symbol": "BINANCE:BTCUSDT",
  "side": "{{strategy.position_size>0? 'SELL':'BUY'}}",
  "qty": "{{calc_position_size}}",
  "sl": "{{strategy.order.price - atr14*sl_mult}}",
  "tp1": "{{strategy.order.price + risk}}",
  "trail": "AI_SUPERTREND"
}

    sl_mult: 0.6 or 0.75 depending on OB height

    qty from dynamic risk‐per‐trade calc

# 5 Back-testing & Optimisation

    Back‑tester: Signals & Overlays on 3 m, 10,000 bars from 2023‑01‑01

    Optimiser: Max PF subject to DD < 8 %

    Walk‑forward: 90‑day slices, maintain PF ≥ 1.4

    Slippage check: Paper‑account dry‑run, ensure slippage < 0.06 %

# 6 Risk & Account Parameters
Parameter	Value
Leverage	20× (ramp 5→10→20)
Margin mode	Isolated
Risk per trade	0.75 % equity
Max intraday loss	2.5 R
Daily circuit‑breaker	−3 R on any direction
Equity‑curve soft‑stop	Disable if equity < (ATH–6 R)
Dynamic risk throttle	Risk% = base×max(0.5, 1–Drawdown/8 %)
Commission	Maker 0.02 %, Taker 0.04 %
Funding filter	Skip entries if funding > 0.06 % 8 h

# 7 Deployment Checklist

Indicator & PAC setup on TradingView

Alert script & JSON template configured

Testnet dry‑run (48 h) at 5× leverage, 0.5 % risk

    Live deploy at 20× leverage once validated

# 8 Maintenance

    Daily: Verify alerts active; funding < 0.06 %; clear logs

    Weekly: Tune Smart‑Trail & PAC OB precision; check latency

    Monthly: 60‑day walk‑forward; Bayesian re‑optimisation (Optuna/Ax)

    Quarterly: Monte‑Carlo stress tests; refresh fee/funding model

##########
Strategy Setup
1  Create / upgrade TradingView workspace

    Already on Premium; ensure chart timezone = Exchange

2  Install LuxAlgo Premium indicators

    Already licensed; add SMC, PAC, Osc Matrix, S&O, AI SuperTrend

3  Configure indicator inputs

    Follow exact recipes above; include PAC settings for BOS, OB, liquidity

4  Build master Alert() script

    Single composite alert per chart; paste JSON template

5  Safety & QA

    Add back‑tester overlay; test–fire alerts; spam guard off

7  Daily launch routine

    Toggle indicators, confirm funding, clear TV logs, start FastAPI

This final version integrates Price Action Concepts™, tightens risk, scales R:R, and preserves your high‑frequency, high‑quality edge. Let me know if anything else needs refinement!






## Strategy Setup

Here’s the updated **Strategy Setup** reflecting **3‑minute charts only** and including **ETHUSDT** and **SOLUSDT**:

---

## Strategy Setup

### 1  Create / Upgrade the TradingView Workspace
- You already have **TradingView Premium**.  
- Set **Chart Timezone** → *Exchange*.  
- Create a **3‑minute multi‑pair layout** with three tabs:  
  1. **BTCUSDT 3 m**  
  2. **ETHUSDT 3 m**  
  3. **SOLUSDT 3 m**  
- Save this layout as **“Scalp 3m Multi‑Pair”**.  
- (Optional) Clone the layout for quick “backup” edits.

---

### 2  Install LuxAlgo Premium Indicators  
*(Add to *all three* 3 m tabs)*  
- **Smart‑Money‑Concepts v1.4**  
- **Price Action Concepts™**  
- **Oscillator Matrix v6.1**  
- **Signals & Overlays v5.3**  
- **AI SuperTrend Clustering v2.0**  

---

### 3  Configure Indicator Inputs (Exact Recipe)  
> Apply these *identical* settings on BTC, ETH and SOL 3 m charts.

#### Smart‑Money‑Concepts v1.4  
| Input               | Value                        |
|---------------------|------------------------------|
| HTF Time‑frame      | 15 min                       |
| LTF Time‑frame      | Chart                        |
| Volumetric OB       | ON; Strength ≥ 30; Dominance ≥ 60% |
| BOS/CHoCH labels    | ON (internal + swing)        |
| Premium/Discount    | ON                           |
| Mitigation logic    | CLOSE                        |

* In detail *

✅ Final Config: LuxAlgo – Smart Money Concepts v1.4
🔹 Real-Time Structure (Top Section)
Setting	Value
Show Internal Structure	✅ ON
Bullish/ Bearish Structure	All
Internal Label Size	tiny
Show Swing Structure	✅ ON
Swing Label Size	small
Show Strong/Weak High/Low	✅ ON
Show Swings Points	(50 is fine)
Confluence Filter	❌ OFF
🔹 Order Blocks
Setting	Value
Internal Order Blocks	✅ ON (5)
Swing Order Blocks	❌ OFF
Order Block Filter	ATR
Order Block Mitigation	High/Low
🔹 Equal Highs/Lows (EQH/EQL)
Setting	Value
Equal High/Low	✅ ON
Bars Confirmation	3
Threshold	0.1
Label Size	tiny
🔹 Fair Value Gaps (FVG)
Setting	Value
Fair Value Gaps	❌ OFF (We don’t need this)
Auto Threshold	✅ ON (can leave checked)
Timeframe	Chart
Extend FVG	1
🔹 Premium / Discount Zones
Setting	Value
Premium/Discount Zones	✅ ON
Premium Zone / Discount Zone / Equilibrium	(Colors optional — leave default or match your theme)
🔹 Input Summary

    Leave “Inputs in status line” ✅ ON (default).

    Do not enable color candles or Historical Mode override.

Once you confirm the above config is applied to all BTCUSDT, ETHUSDT, and SOLUSDT 3m charts



#### Price Action Concepts™  
| Input               | Value                        |
|---------------------|------------------------------|
| BOS/CHoCH           | ON                           |
| Liquidity grabs     | ON                           |
| Order‑Block shading | ON (FVGs + pools)            |
| Equal highs/lows    | ON                           |

* In detail


🧱 Market Structure
Setting	Value
Internal	All (5)
Swing	None
Timeframe	Chart
Show Swing High/Low	❌ Off
Show Strong/Weak HL	❌ Off
Color Candles	❌ Off
📦 Volumetric Order Blocks
Setting	Value
Show Last	✅ ON (5)
Internal Buy/Sell Activity	✅ ON
Show Breakers	❌ OFF
Length	5
Mitigation Method	Close
Timeframe	Chart
Show Metrics	✅ ON
Show Mid‑Line	✅ ON
Hide Overlap	✅ ON
💧 Liquidity Concepts
Setting	Value
Trend Lines	❌ OFF
Patterns	❌ OFF
Show Pattern Zones	✅ ON
Equal H&L	Short-Term only
Liquidity Grabs	✅ ON
🧾 Imbalance Concepts
Setting	Value
Imbalance Type	FVG
Mitigation Method	Close
Timeframe	Chart
Extend Imbalance	10
Volatility Threshold	0
🧠 Premium & Discount Zones
Setting	Value
Premium/Discount Zones	❌ OFF

    We get P/D logic from SMC v1.4, so we leave this version off to avoid redundancy.

📉 Highs & Lows MTF

All settings: ❌ OFF
🔺 Fibonacci Retracements

Leave OFF, not needed for this strategy.
🎨 General Styling
Setting	Value
Internal Label	Tiny
Swing Label	Small
Structures Theme	Colored
EQHL Size	Tiny
OB Metrics Size	Small
Dashboard Location	Top Right
Dashboard Size	Small


#### Oscillator Matrix v6.1  
| Input              | Value            |
|--------------------|------------------|
| Money‑Flow length  | 34; Auto thresh; Overflow ON |
| Hyperwave length   | 14; SMA 3        |
| Divergences        | Realtime; Sens 2 |
| Confluence zones   | ON               |

* In detail
✅ LuxAlgo – Oscillator Matrix v6.1 Configuration
Smart Money Flow
Input	Value
✅ Show Money Flow	ON
Money Flow Length	35
Smooth	6
Thresholds	Auto (colors: Green/Red)
Overflow	✅ ON
HyperWave
Input	Value
✅ Show HyperWave	ON
HyperWave Length	7
Signal	SMA 3
Colors	Green / Gray (80%) opacity
HyperWave Divergences
Input	Value
✅ Show Divergences	ON
Divergence Sensitivity %	20
✅ Show Divergences on Chart	ON
Divergence Colors	Blue / Red
Reversals
Input	Value
✅ Show Reversals	ON
Reversal Factor	5
Reversal Colors	Green / Red
Confluence Zones
Input	Value
✅ Upper Confluence	ON
✅ Lower Confluence	ON
Show Confluence Meter	OFF
Meter Width


#### Signals & Overlays v5.3  
| Input              | Value                 |
|--------------------|-----------------------|
| Mode               | Confirmation          |
| Autopilot          | ON                    |
| Smart‑Trail        | Trail + Direction     |
| Classifier rating  | ≥ 2                   |


* In detail

✅ Signals & Overlays v5.3 Configuration (Exact Settings)
📌 Basic Settings
Setting	Value
Presets/Filters	None (custom setup)
Signal Mode	Confirmation
ML Signal Classifier	OFF
Signals Sensitivity	(Default)
Candle Coloring	As per your preference
📌 Indicator Overlay
Overlay	Enabled	Value
✅ Smart Trail	Yes	3 (both blue/red coloring enabled)
⬜ Reversal Zones	No	—
⬜ Trend Catcher	No*	(You’re using this in alert creator only)
⬜ Trend Tracer	No	—
⬜ Neo Cloud	No	—
📌 TP/SL Settings
Setting	Value
TP/SL Levels	None
Distance	5

    TP/SL logic handled by your FastAPI trade engine; we skip in-chart drawing here.

📌 Advanced Settings
Setting	Value
ML Classifier	1234
Autopilot Sensitivity	Off
📌 Dashboard Settings
Setting	Value
Location	Bottom Right
Size	Small
Color	(Dark blue swatch)
✅ Trend Strength	Yes
⬜ Volatility	No
⬜ Squeeze	No
⬜ Volume Sentiment	No
📌 Custom Alert Creator
Signal Type	Value	Step
Signal	Any Bullish	1
ML Classifier	1234	1
Smart Trail	Price Only	1
Trend Tracer	Uptrend Only	1
Trend Catcher	Uptrend Only	1
Neo Cloud	Uptrend Only	1



#### AI SuperTrend Clustering v2.0  
| Input          | Value                  |
|----------------|------------------------|
| ATR length     | 10                     |
| Factor range   | 1–3 step 0.5           |
| Memory         | 10 trades              |
| Cluster        | BEST                   |

* In detail
✅ Step 5: Configure LuxAlgo – AI SuperTrend Clustering v2.0

    This handles dynamic trailing stop logic. It automatically picks the best ATR-based SuperTrend factor based on recent trade performance. Let’s set it up precisely for your BTC‑Alpha + ETH/SOL Hedge system.

🔧 Inputs Tab – Recommended Settings
Setting	Value
ATR Length	10
Factor Range	1 – 3
Factor Step	0.5
Performance Memory	10
Cluster Selection	BEST

    This means it will dynamically test:

        1.0× ATR

        1.5× ATR

        2.0× ATR

        2.5× ATR

        3.0× ATR
        And pick the best one every 10 trades.

        




---

### 4  Build the Master Alert() Script  
> **Only on the BTCUSDT 3 m tab** (ETH/SOL don’t fire alerts—they’re hedged programmatically).

1. In **Pine Editor**, paste:
   ```pinescript
   alert_message = "{HTF_BULL} AND {OB_DEMAND_HIT} AND {HW_CROSS_UP} AND {NO_OVERFLOW} AND {BUY_CONF} AND {TRAIL_BULL}"
   alert(alert_message, alert.freq_all)
   ```
2. Save as **“BTC_Scalp_LuxSteps”** and **add** to BTC 3 m chart.  
3. Create an **Alert**:
   - **Condition:** BTC_Scalp_LuxSteps → Any alert() call  
   - **Options:** Once per bar  
   - **Webhook URL:** `https://<your-domain>/webhook`  
   - **Message:** see below  

#### Webhook JSON Template
```json
{
  "symbol": "BINANCE:BTCUSDT",
  "side": "{{strategy.position_size>0? 'SELL':'BUY'}}",
  "qty": "{{calc_position_size}}",
  "sl": "{{strategy.order.price - atr14*sl_mult}}",
  "tp1": "{{strategy.order.price + risk}}",
  "trail": "AI_SUPERTREND"
}
```
- `sl_mult` = 0.6 if OB height < 1.2× ATR14, else 0.75  
- `qty` calculated as 0.75% of equity ÷ (Entry–SL)

---

### 5  Safety, Spam‑Proofing & QA
- **Back‑test overlay:** Attach “Signals & Overlays Back‑tester” on BTC 3 m.  
- **Spam Guard:** Disable pop‑up & email notifications.  
- **Test Fire:** “Send test notification” → confirm success.  
- **Economic Filter:** High‑impact events blocked server‑side.

---

### 7  Daily Launch Routine (Operator Cheat‑Sheet)
1. **Confirm indicators** (SMC, PAC, Osc, S&O, AI ST) are enabled on all three 3 m tabs.  
2. **Verify** BTC alert is *active* (green).  
3. Check **Binance funding rate** < 0.06% (via CoinGlass/API).  
4. Clear TV **notification log**.  
5. Start/reset FastAPI **after** alerts live.  
6. Monitor **latency** & **error logs** (CloudWatch/Slack).

---

With this, your **3 m multi‑pair** workspace is fully configured for **BTC‑alpha + dynamic ETH/SOL hedge**, complete with **Price Action Concepts™** precision on all three charts.



















###### Profit Analysis
Let's break down the potential earnings clearly and realistically, considering your initial capital of **1,000 USDT**:

### Key Strategy Metrics (given):
- **Win Rate**: ~65% (average between 62%–68%)
- **Profit Factor (PF)**: ≥ 1.6
- **Risk per trade**: 0.75% of capital (as given: `Account_Equity × 0.0075`)
- **Leverage**: 20×
- **Commission**: Maker 0.02%, Taker 0.04% (average ~0.03%)
- **Typical Reward (R)**: 1 R per trade at TP-1 (partial profit-taking)

---

## Step-by-step calculation:

### 1. Risk per trade:
- Risk per trade = `1000 USDT × 0.0075 = 7.5 USDT`.

### 2. Win/Loss scenario (per trade):
- **Winning trade** (1R):
  - Gross profit ≈ 7.5 USDT
  - Less commission and slippage (~0.06% combined, standard slippage + fees):
    - Approximate total fees = `(trade value × 0.06%) × 2 (entry+exit)`
    - Trade value (Notional) ≈ Risk × Leverage = 7.5 USDT × 20 = **150 USDT**
    - Fees ≈ `150 × 0.0006 × 2 = 0.18 USDT`
  - Net profit per winning trade ≈ `7.5 - 0.18 ≈ 7.32 USDT`

- **Losing trade** (-1R):
  - Gross loss = -7.5 USDT
  - Fees similarly calculated (~0.18 USDT)
  - Total loss per losing trade ≈ `7.5 + 0.18 ≈ -7.68 USDT`

### 3. Average profit per trade based on win-rate:
- Win-rate ≈ 65% (Win: 0.65, Loss: 0.35)
- Expected value per trade:
\[
= (0.65 × 7.32) - (0.35 × 7.68) \\
= 4.758 - 2.688 \\
= 2.07\text{ USDT per trade (expected value)}
\]

Thus, **each trade (on average) makes about 2.07 USDT net**.

---

## 4. Estimating Daily Earnings:
- **Frequency**: Scalping (3m/5m chart), typically ~5–10 signals/day realistically; let's say **7 signals/day**.
- Expected daily earning:
\[
= 7 \text{ trades} × 2.07 \text{ USDT per trade} \\
≈ 14.49 \text{ USDT/day}
\]

---

## 5. Estimating Monthly Earnings (Assuming 22 Trading Days):
- Expected monthly earnings:
\[
= 14.49 × 22 \\
≈ 318.78 \text{ USDT/month}
\]

---

## 6. Projected Monthly Return on Capital:
- Monthly Return ≈ `(318.78 / 1000) × 100% ≈ 31.88%`
- Annualized (without compounding) ≈ `31.88% × 12 ≈ 382.56%`

---

## ✅ **Realistic Earnings Estimate (Summary)**:
| Timeframe | Approx. Earnings | % Return |
|-----------|------------------|-----------|
| Daily     | ~14.49 USDT      | 1.45%     |
| Monthly   | ~318.78 USDT     | 31.88%    |
| Yearly    | ~3825.60 USDT    | 382.56%   |

---

## ⚠️ **Important Considerations**:
- **Slippage & market conditions:** The above calculations assume relatively ideal market conditions.
- **Execution speed & latency:** Actual performance may vary depending on server latency and API response.
- **Variance & drawdown:** Be prepared for drawdowns; daily losses can occur.

---

**Conclusion:**
With a 1,000 USDT initial capital, you can realistically target around **14.50 USDT/day** or about **319 USDT/month** after considering commissions and slippage, provided the strategy is executed with precision and stable market conditions prevail.







###### AWS Server Notes
Action | Latency Range
Binance WebSocket update | ~10–50 ms
Redis read/write | ~1 ms
PostgreSQL local write (single row) | ~3–10 ms
Trade API request (internal) | ~15–30 ms
Binance Futures API order (external) | ~50–150 ms


➡️ Total round-trip latency from receiving a price to placing a trade can be under 150–250ms, depending on:
    your order.post() to Binance
    Binance response speed (usually fast in Tokyo region)
    internal optimization (no blocking code, async everywhere)

