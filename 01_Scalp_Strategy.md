# 01_Scalp_Strategy.md – BTC-USDT Binance Futures (Isolated 20×)

## Scope
Automated high-frequency scalping bot driven by LuxAlgo Premium indicators on TradingView, executed via FastAPI webhooks to Binance Futures. Built for 3 m/5 m charts, targeting 62-68 % win-rate, PF ≥ 1.6, ≤ 2.5 R day-draw-down.

---

## 0 Prerequisites

| Item                         | Notes                                                 |
|------------------------------|-------------------------------------------------------|
| **TradingView Premium**      | Real-time data + unlimited alerts                     |
| **LuxAlgo Premium Suite**    | • Smart-Money-Concepts v1.4 (Apr 2025)<br>• Signals & Overlays v5.3<br>• Oscillator Matrix v6.1<br>• AI SuperTrend Clustering v2.0 |
| **Binance Futures Account**  | Isolated margin; 20× max leverage; fee tier Maker 0.02 %, Taker 0.04 % |
| **FastAPI Trade-Router**     | Expects JSON: `{symbol, side, qty, sl, tp1, trailing}` |
| **Economic-calendar API**    | Block trading ±10 min around high-impact events       |

---

## 1 Indicator Configuration

### 1.1 Smart-Money-Concepts (SMC)
- **HTF Time-frame:** 15 minute
- **LTF Time-frame:** chart (3 m or 5 m)
- **Volumetric Order Blocks:** ON (Strength ≥ 30, Buy/Sell dominance ≥ 60 %)
- **BOS/CHoCH labels:** ON (internal + swing)
- **Liquidity grabs:** ON (display only)
- **Premium/Discount zones:** ON (equilibrium shading)
- **Mitigation logic:** CLOSE

### 1.2 Oscillator Matrix
- **Money-Flow:** length 34, thresholds auto, overflow ON
- **Hyperwave:** length 14, signal SMA 3
- **Divergences:** realtime, sensitivity 2
- **Confluence zones + meter:** ON

### 1.3 Signals & Overlays
- **Mode:** Confirmation
- **Autopilot:** ON (sensitivity optimiser)
- **Smart-Trail:** Trail + Direction filter
- **Trend Catcher:** ON (entry confidence)
- **Classifier:** Use only ratings ≥ 2

### 1.4 AI SuperTrend Clustering (trailing)
- **ATR length:** 10
- **Factor range:** 1–3, step 0.5
- **Performance memory:** 10
- **Cluster:** BEST

---

## 2 Trading Logic (LONG side)
1. **Context filter:**
   - 15 m internal structure bullish AND price in discount half of 15 m range
   - Money-Flow above lower threshold OR overflow recently ended
   - Funding-rate < 0.06 %

2. **Trigger sequence (Lux steps alert):**
   - Step 1: Price taps Volumetric Demand OB (≥ 60 % buy)
   - Step 2: Hyperwave crosses up below 50-line inside OB
   - Step 3: No active overflow
   - Step 4: Signals & Overlays print Buy Confirmation with bullish Smart-Trail

3. **Entry:** Market (or limit in last 50 % of OB) next candle

4. **Initial Stop-loss:** OB-low – 0.75 × ATR14

5. **Position size:**  
   `USDT_qty = (Account_Equity × 0.0075) / (Entry – SL)`  
   `Notional = USDT_qty × 20 (leverage)`

6. **Trade management:**
   - TP-1: +1 R close 50 %, SL→BE
   - AI-SuperTrend trail remainder
   - Hard exit on opposite CHoCH or 30 min timeout

7. **Daily circuit-breaker:** Disable after −3 R realised PnL

*SHORT rules mirror LONG side.*

---

## 3 Alert-Scripting (TradingView)
```plaintext
// Placeholders
{HTF_BULL} AND {OB_DEMAND_HIT} AND {HW_CROSS_UP} AND {NO_OVERFLOW} AND {BUY_CONF} AND {TRAIL_BULL}
```
- Create single **Any alert() call** → Webhook URL (FastAPI).

### 3.1 Webhook JSON template
```json
{
  "symbol": "BINANCE:BTCUSDT",
  "side": "BUY",
  "qty": "{{calc_position_size}}",
  "sl": "{{strategy.order.price - atr14*0.75}}",
  "tp1": "{{strategy.order.price + risk}}",
  "trail": "AI_SUPERTREND"
}
```

---

## 4 Back-testing & Optimisation
1. Add **Signals & Overlays Back-tester** to 3 m chart
2. Bars = 10,000, Start = 2023-01-01
3. External filter = Osc Matrix Money-Flow positive
4. Optimiser → Max PF with DD < 8 %
5. Walk-forward 90-day slices; maintain PF ≥ 1.4
6. Deploy to paper-account; compare slippage (< 0.06 %)

---

## 5 Risk & Account Parameters

| Parameter            | Value                        |
|----------------------|------------------------------|
| Margin mode          | Isolated                     |
| Leverage             | 20× (ramp-up: 5× → 10× → 20×) |
| Max intraday loss    | 2.5 R                        |
| Daily circuit breaker| −3 R                         |
| Commission           | Maker 0.02 %, Taker 0.04 %    |
| Funding filter       | Skip if funding > 0.06 % 8h  |

---

## 6 Deployment Checklist
- [ ] Import indicators to TradingView
- [ ] Configure master alert script
- [ ] Verify FastAPI JSON parsing
- [ ] 48-hour dry-run on testnet
- [ ] Live deploy at 5× leverage, 0.5 % risk

---

## 7 Maintenance
- **Weekly:** Optimise Smart-Trail sensitivity
- **Monthly:** 6-month walk-forward analysis
- **Quarterly:** Review funding-rate and fee tier impact

---






























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





##########
Strategy Setup
#########

Below is a practical, point‑by‑point checklist that takes you from a fresh TradingView account to a fully armed, one‑click LuxAlgo “Any alert()” that can later be wired to your FastAPI router. I keep every step self‑contained and explain *why* you do it, so you understand how each switch supports the strategy you drafted.

---

## 1  Create / upgrade the TradingView workspace

| Step | Action | Why it matters |
|------|--------|----------------|
|1.1|Sign‑in → **Upgrade** → choose **Premium** plan|Premium unlocks real‑time crypto feeds, second‑interval replay, unlimited indicators & alerts—mandatory for high‑frequency scalping.|
|1.2|Open **Chart Layout** → gear‑icon **Settings** → **Timezone** = *Exchange*|Keeps indicator timestamps identical to Binance; avoids desync when funding windows flip.|
|1.3|Add two tabs inside the same layout:<br>• **BTCUSDT 3 m**<br>• **BTCUSDT 5 m**|Your bot toggles both; separate tabs let you launch alerts on each time‑frame without re‑configuring inputs.|

---

## 2  Install LuxAlgo Premium indicators

You must be granted Premium access by the LuxAlgo bot after purchase. Once that’s done:

| Step | Action | Why |
|------|--------|-----|
|2.1|On the 3 m chart press **Indicators** → “Invite‑only scripts”|Shows all licensed LuxAlgo modules.|
|2.2|Add **LuxAlgo – Smart Money Concepts v1.4**|SMC supplies HTF structure, BOS/CHoCH, order blocks—core context filter (#2).|
|2.3|Add **LuxAlgo – Signals & Overlays v5.3**|Generates confirmation/Smart‑Trail events—acts as entry trigger (#2‑Step 4).|
|2.4|Add **LuxAlgo – Oscillator Matrix v6.1**|Money‑Flow, Hyperwave & divergences—feeds Step 2 and overflow logic.|
|2.5|Add **LuxAlgo – AI SuperTrend Clustering v2.0**|Dynamic trailing stop that self‑optimises—automates exit management.|

> **Tip:** Repeat the same adds on the 5 m tab so both charts stay in sync.

---

## 3  Configure indicator inputs (exact recipe)

> Use the *cog* (⚙️) next to each script name. Only inputs mentioned below change—leave the rest default.

### 3.1  Smart Money Concepts v1.4  
| Input | Value | Why |
|-------|-------|-----|
|HTF Time‑frame|**15 minute**|Matches “context filter” spec (macro structure).|
|LTF Time‑frame|**Chart**|Locks to active tab (3 m/5 m).|
|Volumetric Order Blocks|**ON**; Strength ≥ 30; Dominance ≥ 60 %|Filters weak OBs; ≥ 60 % buy or sell dominance lines up with trigger Step 1.|
|BOS/CHoCH labels|**ON** (internal + swing)|Bot must spot opposite CHoCH to force exit.|
|Liquidity grabs|**ON (display only)**|Visual sanity check; not in automation.|
|Premium/Discount zones|**ON**|Needed to verify “price in discount half”.|
|Mitigation logic|**CLOSE**|Declares OB mitigated only after close beyond — avoids premature deletion.|

### 3.2  Oscillator Matrix v6.1  
| Input | Value | Why |
|-------|-------|-----|
|Money‑Flow length|**34**|Slower; reduces noise on 3 m bars.|
|Thresholds|**Auto**|Adapts to regime changes.|
|Overflow|**ON**|We exit/skip if overflow active.|
|Hyperwave length|**14**, **Signal SMA 3**|Matches strategy for cross.|
|Divergences sensitivity|**2**|Mid‑sensitivity to avoid spam.|
|Confluence zones + meter|**ON**|Extra heads‑up for manual oversight.|

### 3.3  Signals & Overlays v5.3  
| Input | Value | Why |
|-------|-------|-----|
|Mode|**Confirmation**|Generates bullish/bearish “Buy Confirmation” events (not raw buy/sell).|
|Autopilot|**ON**|Lets Lux optimiser nudge sensitivity by volatility—keeps win‑rate stable.|
|Smart‑Trail|**Trail + Direction filter**|Merges trend filter with trailing stop (no need for extra EMA).|
|Trend Catcher|**ON**|Extra gate on entry—only when both algos agree.|
|Classifier minimum rating|**≥ 2**|Ignores low‑confidence signals.|

### 3.4  AI SuperTrend Clustering v2.0  
| Input | Value | Why |
|-------|-------|-----|
|ATR length|**10**|Shorter ATR suits 3 m bars.|
|Factor range|**1 – 3** in 0.5 steps|Creates six variants for ensemble.|
|Performance memory|**10**|Uses last 10 trades to rank which factor performs best now.|
|Cluster|**BEST**|Applies only best‑performing SuperTrend to trailing.|

---

## 4  Build the master Alert() script

TradingView needs **one** composite alert per chart so every qualifying bar sends a webhook payload.

### 4.1 Insert Lux “Any alert()” line

1. In **Pine Editor** (bottom panel), start a blank script.  
2. Paste exactly one line:

```pinescript
alert_message = "{HTF_BULL} AND {OB_DEMAND_HIT} AND {HW_CROSS_UP} AND {NO_OVERFLOW} AND {BUY_CONF} AND {TRAIL_BULL}"
alert(alert_message, alert.freq_all)
```

*Why*  
- `alert.freq_all` fires on *every tick*, not just on close—critical for scalp latency.  
- Each curly placeholder corresponds to the LuxAlgo built‑in *steps* you defined in the chart UI (Lux docs → “Steps alerts”). Internally, Lux expands them to 1/0.  

> **Tip:** Save script as **“BTC_Scalp_LuxSteps”**; add to both charts.

### 4.2 Create the TradingView alert

| Screen | Setting | Value / Notes |
|--------|---------|---------------|
|**Condition**|Your script → **Any alert() function call**|Ensures the exact same logic for long *and* short; side is chosen later in webhook.|
|**Options**|**Once per bar**|Stops duplicate hits inside the same candle but still fires immediately.|
|**Expiration**|Open‑ended|You’ll deactivate via code when circuit‑breaker triggers.|
|**Webhook URL**|Your FastAPI endpoint (add later)|Holds JSON POST.|
|**Alert message**|Paste the **JSON template** below.|TradingView replaces the curly tags at runtime.|

#### Webhook JSON template (copy into *Message* box)
```json
{
  "symbol": "BINANCE:BTCUSDT",
  "side": "{{strategy.position_size>0? 'SELL':'BUY'}}",
  "qty": "{{calc_position_size}}",
  "sl": "{{strategy.order.price - atr14*0.75}}",
  "tp1": "{{strategy.order.price + risk}}",
  "trail": "AI_SUPERTREND"
}
```
**What for**  
- `{{calc_position_size}}` is a Pine variable you’ll output in the same script once you wire account‑equity input. For now it can be a placeholder.  
- The ternary operator flips *side* automatically so one alert covers both directions.  
- Values like `atr14` are fetched from your script’s runtime vars.

---

## 5  Safety, spam‑proofing & QA

| Check | How | Why |
|-------|-----|-----|
|Back‑test overlay|Add **“Signals & Overlays Back‑tester”** to 3 m tab, 10 k bars, start = 2023‑01‑01|Confirms the alert() script sees the same events back‑tester counts.|
|Alert spam guard|Set **“max alerts”** in account to e‑mail + pop‑up = off|You rely exclusively on webhook; reduce noise.|
|Test fire|With webhook blank, click **Send test notification** → see if TradingView says “success”|Validates JSON formatting before pointing at live FastAPI.|
|Economic calendar filter|TradingView’s built‑in calendar can’t auto‑snooze alerts. You’ll block in the server layer via the API later|Keeps chart side lightweight.|

---

## 6  Cloning the setup to the 5 m tab

1. Right‑click chart tab → **Clone**.  
2. Change Interval to **5**.  
3. Verify SMC LTF automatically picks “Chart”; no other inputs change.  
4. Re‑open Alerts panel, duplicate the alert, name it **BTC_Scalp_5m**—everything else identical.

---

## 7  Daily launch routine (operator cheat‑sheet)

1. **Toggle visibility** of all four Lux indicators → verify green check marks.  
2. Make sure alert dot in right toolbar is *green* for both charts.  
3. Confirm Binance funding‑rate < 0.06 % (e.g., CoinGlass) before market open; disable alerts if breached.  
4. Clear TradingView notification log—easier to spot misfires.  
5. Start your FastAPI router **after** alerts are live (guarantees endpoint is listening).

---

### You’re now ready for the FastAPI & server‑side leg.

The above gives you a stable, version‑locked TradingView + LuxAlgo environment that emits clean, atomic webhook calls matching every single requirement in your .md spec. When you’re ready to wire the Python router, just point the webhook URL to it and parse the JSON you already embedded.


##########
##########
##########