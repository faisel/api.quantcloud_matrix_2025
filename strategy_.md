Below is a concise, no-fluff playbook that ties together the three LuxAlgo suites you listed and keeps the focus squarely on risk and win-rate. Treat it as a starting template—forward-test and tweak the parameters that appear in **bold** until they match your temperament and the current BTCUSDT micro-structure.

---

## 1 · Framework & Chart Setup  

| Layer | Time-frame | Purpose | Tool & Settings |
|-------|-----------|---------|-----------------|
| **HTF Bias** | 15 m | Trade direction filter | **Trend Catcher** → only take longs when green, shorts when red. |
| **Execution** | 1 m or 3 m | Precise scalp entries | Price-Action Concepts + Signals & Overlays. |
| **Confirmation** | same as execution | Avoid false breaks | Oscillator Matrix panel. |

> **Why two time-frames?** The 15 m cleans out the noise so you skip fading a strong intraday trend; the sub-5 m chart lets you keep stops tiny relative to 20× leverage.

---

## 2 · Entry Logic (confluence-based)

### Long Setup  

1. **HTF filter**  
   - 15 m Trend Catcher **green** AND Trend Strength ≥ **10**.

2. **Liquidity sweep + structure shift** on the 1 m/3 m  
   - Price sweeps an **Equal Low (EQL)** or tagged support, then prints a bullish **CHoCH+** or bullish **BOS** (close above last swing-high).  
   - Candle closes inside or just above the swept zone.

3. **Oscillator confirmation** (any two)  
   - **HyperWave** flips bullish (line slopes up, colour change).  
   - **Smart Money Flow** > 0.  
   - Bullish **Reversal Signal** dot printed.

4. **Trigger**  
   - Enter at close of the confirmation candle OR set buy-stop **0.1 %** above its high to avoid late fills.

### Short Setup  

Mirror the above:

- Trend Catcher red, Trend Strength ≥ 10.  
- Sweep of **Equal High** / resistance, bearish CHoCH+ or BOS down.  
- HyperWave bearish, Smart Money Flow < 0, or red Reversal dot.  
- Sell at close or sell-stop **0.1 %** below the candle low.

---

## 3 · Risk Management Rules (the hard line)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Max risk / trade | **0.5 %** equity (at 20× ≈ 10 % position move) | Prevents a string of losses wiping you out. |
| Stop-loss | Below/above sweep wick **or 1.2 × ATR(14) on exec TF**—whichever is tighter. |
| Take-profit | **2 ×** stop distance (fixed 1 : 2 RR). |
| Daily loss cap | **2 %** of account or **3** consecutive losses—stop trading. |
| Daily target | When equity grows **3 %** in a session, stand down—protect the win-rate. |

*Tip:* Hit 1 R quickly? Scale out 50 %, move stop to breakeven and let the rest hunt the full 2 R.

---

## 4 · Pine Script Alert Template (TradingView v5)

> Replace the placeholder functions with the actual LuxAlgo output names visible in your Data Window.  

```pinescript
//@version=5
strategy("BTCUSDT LuxAlgo Scalp 20x", overlay=true,
     default_qty_type=strategy.percent_of_equity,
     default_qty_value=0.5,   // 0.5 % per trade
     margin_long=20, margin_short=20)

///////// INPUTS
rr      = input.float(2.0,  "Risk-Reward", step=0.1)
atrMult = input.float(1.2,  "ATR Multiplier", step=0.1)
///////// INDICATORS (pseudo names)
tcDir       = lux_tc_dir()          //  1 = green, -1 = red
tcStrength  = lux_tc_strength()     //  0-100
bosBull     = lux_bos_bullish()     //  true/false
bosBear     = lux_bos_bearish()
chochPlusUp = lux_choch_plus_bull()
chochPlusDn = lux_choch_plus_bear()
hyperBull   = lux_hyper_bull()
hyperBear   = lux_hyper_bear()
smfVal      = lux_smf()             //  >0 bull, <0 bear
revBull     = lux_rev_bull()
revBear     = lux_rev_bear()

///////// CONDITIONS
longCond  = tcDir==1 and tcStrength>=10 and (bosBull or chochPlusUp) and
           (hyperBull and (smfVal>0 or revBull))

shortCond = tcDir==-1 and tcStrength>=10 and (bosBear or chochPlusDn) and
           (hyperBear and (smfVal<0 or revBear))

///////// ENTRY + RISK
atr   = ta.atr(14)
risk  = atr*atrMult
if longCond
    stop = close - risk
    target = close + risk*rr
    strategy.entry("Long", strategy.long)
    strategy.exit("TP/SL Long", "Long", stop=stop, limit=target)

if shortCond
    stop = close + risk
    target = close - risk*rr
    strategy.entry("Short", strategy.short)
    strategy.exit("TP/SL Short", "Short", stop=stop, limit=target)
```

Create an **alert** on this strategy with the JSON you pass to your webhook, for example:

```json
{
  "side": "{{strategy.order.action}}",
  "ticker": "{{ticker}}",
  "price": "{{close}}",
  "size": "{{strategy.order.contracts}}",
  "sl": "{{strategy.order.exit_price_1}}",
  "tp": "{{strategy.order.exit_price_2}}",
  "time": "{{time}}"
}
```

---

## 5 · Optimisation Checklist

| Step | What to tune | Typical range |
|------|--------------|---------------|
| Execution TF | 1 m vs 3 m | Faster gives more trades, but worse spread-slippage. |
| Trend Strength threshold | 5-20 | Higher = fewer, cleaner signals, higher win-rate. |
| ATR multiplier | 1.0-1.5 | Lower = tighter stops → higher RR but lower hit-rate. |
| Max trades / day | 3-10 | Controls over-trading during chop. |

Run TradingView’s built-in **Strategy Tester** on at least 90 days of BTCUSDT 24/7 data; forward-test live in “paper-trading” for two weeks before risking capital.

---

### Final Thoughts

- **Respect the daily loss cap**—it protects both equity and psychology.  
- Avoid entries in the 2 minutes before/after high-impact macro news; slippage kills scalps.  
- Keep the webhook as stateless as possible: every alert carries full position, SL, and TP so your trade bot can recover from disconnects.  
- LuxAlgo prints lots of signals; the *edge* comes from stacking them and sitting out everything else.

Trade safe, iterate often, and may your win-rate stay north of 60  %.