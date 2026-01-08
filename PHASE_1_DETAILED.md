# 📋 PHASE 1 INSTRUCTIONS - Logic Discovery & Clarification

**User-focused guide. You decide what to tell the AI.**

---

## 🎯 WHAT IS PHASE 1?

AI #1 will understand your trading strategy completely by:
- Asking you questions about your trading logic
- Understanding your entry/exit rules
- Learning your risk management approach
- Clarifying any ambiguous points
- Getting your approval before moving forward

**Duration:** 2-4 hours  
**Output:** PHASE_1_APPROVED.txt (AI's understanding of your strategy)

---

## 📝 WHAT YOU NEED TO PREPARE

You have several options. Choose what's easiest for you:

### Option A: You Have a Current EA (Existing Code)
If you already have an EA that works (in MetaTrader):
- [ ] Have your EA file (.mq4, .mq5, or code)
- [ ] Can describe what it does
- [ ] Have backtest results you want to replicate

### Option B: You Have a Trading Journal
If you manually trade or backtested:
- [ ] Have a journal/log of trades
- [ ] Can describe your entry/exit logic
- [ ] Know your risk management rules

### Option C: You Have Just the Strategy
If you know your strategy but haven't coded it:
- [ ] Know your entry conditions
- [ ] Know your exit conditions
- [ ] Know your risk rules

### Option D: You Have Backtest Data/Screenshots
If you have evidence of your strategy working:
- [ ] Screenshots of backtest results
- [ ] Excel file with trade history
- [ ] MT4/MT5 backtest report
- [ ] Journal entries with trades

---

## 🎬 WHAT TO DO IN PHASE 1

### Step 1: Start the AI in OpenHands

Use this prompt (or similar - AI is flexible):

```
I want to build an MQL5 Expert Advisor for my trading strategy.

Here's what I trade:

[Describe your strategy here - be as detailed as possible]

MY ENTRY LOGIC:
- [What makes you enter a trade?]
- [Any indicators you use?]
- [Any time restrictions?]
- [Any other conditions?]

MY EXIT LOGIC:
- [How do you exit winning trades?]
- [How do you exit losing trades?]
- [Do you use trailing stops?]
- [Any time-based exits?]

MY RISK MANAGEMENT:
- [How much do you risk per trade?]
- [How do you calculate lot size?]
- [Maximum positions open?]
- [Daily loss limits?]

SPECIAL CONDITIONS:
- [Any other rules or restrictions?]
- [Best times to trade?]
- [Worst times to trade?]

Please ask me clarification questions until you understand 
my strategy 100%, then create PHASE_1_APPROVED.txt 
documenting everything.
```

### Step 2: Answer AI's Questions Thoroughly

The AI will ask questions like:
- "What exactly do you mean by this condition?"
- "Is this a required condition or optional?"
- "What values do you use here?"
- "Are there any exceptions to this rule?"
- "Can you give me an example?"

**Your job:** Answer completely and honestly. Don't be vague.

### Step 3: Provide Evidence (Optional but Helpful)

You can upload/describe:
- [ ] Backtest results showing your strategy works
- [ ] Screenshots of winning trades
- [ ] Your current EA code (if you have one)
- [ ] Trading journal entries
- [ ] Screenshots of indicators on chart
- [ ] Excel file with trade statistics

The AI will analyze this and ask follow-up questions.

### Step 4: Confirm Understanding

When the AI says "Is my understanding correct?", respond clearly:
- ✅ "Yes, that's exactly right"
- ❌ "No, here's what's wrong..."
- ⚠️ "Mostly correct, but also..."

Don't move forward until you're 100% satisfied.

---

## 💡 WHAT TO TELL THE AI (Detailed Examples)

### Example 1: Simple Strategy

```
I trade EURUSD on 1-hour timeframe.

ENTRY:
- RSI(14) crosses above 30 (oversold reversal)
- Price is above 200-period MA (uptrend)
- Time is between 08:00-16:00 London
- Volume is above average

EXIT:
- Profit target: +100 pips
- Stop loss: -50 pips
- Time exit: close at 16:00

RISK:
- 0.01 lot per trade
- Maximum 3 open positions
- Daily loss limit: 2% of account

SPECIAL:
- Skip trading on Fridays
- If I get 2 losses in a row, wait 1 hour before next trade
```

### Example 2: Complex Strategy

```
I trade XAUUSD (Gold) using multiple timeframes.

ENTRY CONDITIONS (ALL must be true):
1. H4 timeframe: Price above 200MA (long-term trend)
2. H1 timeframe: RSI(14) < 30 (oversold)
3. M15 timeframe: MACD positive crossover
4. NOT during US trading hours (too volatile)
5. Stochastic(14) < 20 (confirmation)
6. Time: 06:00-14:00 GMT only

EXIT CONDITIONS:
1. Take Profit: 
   - If risk = $100, profit target = $200 (2:1 ratio)
   - Calculated from entry price + 200 pips
2. Stop Loss:
   - Below recent swing low or -100 pips (whichever is tighter)
3. Trailing Stop:
   - Activate after +50 pips profit
   - Trail by 30 pips
4. Time Exit:
   - Close all at 14:00 GMT
   - Close all on Friday at 12:00

RISK MANAGEMENT:
1. Account size: $10,000
2. Risk per trade: $100 maximum
3. Lot calculation: (Risk / Points) / 100
   - Example: ($100 / 100 pips) / 100 = 0.01 lot
4. Maximum open positions: 2
5. Daily loss limit: 5% = $500
   - If I lose $500 today, stop trading
6. Weekly loss limit: 10% = $1000

SPECIAL CONDITIONS:
1. News avoidance: Skip during high-impact news
2. Correlation: Skip if USDJPY is too volatile
3. Recovery protocol: 
   - After 1 loss: wait 30 minutes
   - After 2 losses: wait 1 hour
   - After 3 losses: stop for the day
4. Best performance: 08:00-12:00 GMT
5. Avoid: 13:00-14:00 (lunch volatility spike)
```

### Example 3: Current EA (If you have code)

```
I have an EA that I want to replicate/improve.

WHAT IT DOES:
- Trades EURUSD H1
- Uses RSI and Bollinger Bands
- Takes profit at 100 pips, stop at 50 pips
- Uses 0.01 lot size

BACKTEST RESULTS:
- 100 trades tested
- 65 wins, 35 losses (65% win rate)
- Total profit: $2,500
- Best trade: +250 pips
- Worst trade: -200 pips
- Average trade: +25 pips

ISSUES I WANT TO FIX:
- Too many losses on Fridays
- Not enough entries on trending days
- Need better risk management

MY ADDITIONS:
- Add time filter: skip Fridays after 14:00
- Add trend confirmation: only trade in direction of 200MA
- Add daily loss limit: stop if -5% reached
```

---

## ✅ PHASE 1 CHECKLIST

Before Phase 1 starts:
- [ ] You know your entry logic (or have EA/journal to reference)
- [ ] You know your exit logic
- [ ] You know your risk management
- [ ] You have OpenHands running
- [ ] Gemini 1.5 Pro is active (Phase 1 AI)
- [ ] You're ready to answer questions

During Phase 1:
- [ ] You answer AI's questions completely
- [ ] You provide examples when asked
- [ ] You confirm understanding multiple times
- [ ] You're 100% satisfied before moving forward

After Phase 1:
- [ ] File PHASE_1_APPROVED.txt exists
- [ ] Contains your strategy fully documented
- [ ] You've reviewed and approved it
- [ ] Ready to move to Phase 2

---

## 🎯 YOUR DELIVERABLE FROM PHASE 1

When Phase 1 is complete, you get:

**PHASE_1_APPROVED.txt** containing:

```
PHASE 1 APPROVAL DOCUMENT
=========================

STRATEGY NAME: [Your strategy name]
USER APPROVAL: [Date/time]
CONFIDENCE: [100% understood]

ENTRY LOGIC:
[Complete, detailed entry conditions]

EXIT LOGIC:
[Complete, detailed exit conditions]

RISK MANAGEMENT:
[Complete risk rules]

SPECIAL CONDITIONS:
[All edge cases and special rules]

EXAMPLES:
[Sample trades demonstrating the logic]

QUESTIONS ASKED AND ANSWERED:
[All clarifications made during Phase 1]

STATUS: READY FOR PHASE 2 ✓
```

---

## 🚀 HOW TO TELL THE MASTER AI YOU'RE DONE

When Phase 1 is complete, create a file:

**File: PROGRESS_UPDATE.txt**

```
PHASE 1 COMPLETE
================

Date Completed: [Today's date]
Duration: [How many hours spent]
AI Used: Gemini 1.5 Pro (Key #1)

WHAT WAS ACCOMPLISHED:
- AI understood my trading strategy
- Clarification questions asked and answered
- PHASE_1_APPROVED.txt created
- All logic documented

DELIVERABLE CREATED:
✓ PHASE_1_APPROVED.txt

STATUS: Ready for Phase 2
NEXT: Switch to DeepSeek and start Phase 2

NOTES:
[Any issues encountered? Any insights?]
```

---

## 💬 WHAT THE AI CAN FREELY DO IN PHASE 1

**Don't limit the AI.** Let it:
- Ask as many questions as needed
- Dig deep into your logic
- Challenge unclear points
- Request examples
- Suggest improvements
- Point out gaps in your strategy
- Identify potential issues
- Question your assumptions

The AI knows how to understand trading strategies. Trust it to ask the right questions.

---

## ⚠️ COMMON ISSUES & SOLUTIONS

**Issue: "I'm not sure how to explain my strategy"**
Solution: Just start talking. Describe one trade example step-by-step. The AI will ask questions.

**Issue: "The AI is asking too many questions"**
Solution: This is GOOD! The more questions, the better it understands. Answer them completely.

**Issue: "I don't have exact numbers for everything"**
Solution: Describe approximately. Say "Around 100 pips" or "roughly 0.01 lot". The AI will ask for precision where needed.

**Issue: "My strategy changes based on market conditions"**
Solution: Explain this! Say "I use X rule normally, but use Y rule during high volatility". The AI will capture this.

---

## 📞 NEED HELP?

If you get stuck:
1. Ask the AI directly: "I'm confused about X, can you help?"
2. Provide more examples
3. Share screenshots or data
4. Describe your current EA code
5. Share your trading journal

The AI is there to help you clarify, not to judge.

---

**NEXT STEP: Start Phase 1! Go to OpenHands and paste your strategy description.**

**Remember: This is just clarification. AI asks questions. You answer. That's it!**

Good luck! 🚀
