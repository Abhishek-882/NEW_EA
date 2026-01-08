# 📋 PHASE 2 INSTRUCTIONS - Logic Documentation

**User-focused guide. Let AI create the specification.**

---

## 🎯 WHAT IS PHASE 2?

AI #2 will take your approved strategy and create a detailed English specification (pseudocode) that:
- Documents every rule in plain English
- Includes exact values and conditions
- Shows examples from your backtest
- Creates the blueprint for coding

**Duration:** 1-2 hours  
**Input:** PHASE_1_APPROVED.txt (from Phase 1)  
**Output:** LOGIC_SPECIFICATION.md (your strategy as detailed documentation)

---

## 📝 WHAT YOU NEED TO DO

### Step 1: Switch to DeepSeek AI

Stop Gemini container, start DeepSeek container.

Use the AI_SWITCHING_GUIDE.md for exact commands.

### Step 2: Upload Context File

In OpenHands, ask the AI:

```
Read the file PHASE_1_APPROVED.txt

This contains my trading strategy that was approved in Phase 1.

Now you are Phase 2. Your job is to create a detailed 
English specification (no code, just clear documentation) 
that describes EXACTLY how my strategy works.

I want the specification to be so detailed that:
- Anyone could understand my strategy
- A programmer could code it without asking questions
- Every rule is explicit with exact numbers
- Examples are provided

Create: LOGIC_SPECIFICATION.md

The specification should include:

1. ENTRY LOGIC SECTION
   - Detailed explanation of each entry condition
   - Exact values and thresholds
   - Examples from my backtest showing the logic working

2. EXIT LOGIC SECTION
   - Take profit logic (exact calculation)
   - Stop loss logic (exact calculation)
   - Trailing stop logic (if applicable)
   - Time-based exits (if applicable)

3. RISK MANAGEMENT SECTION
   - Lot size formula (exact calculation)
   - Position sizing rules
   - Daily/weekly limits
   - Account protection rules

4. SPECIAL CONDITIONS SECTION
   - Edge cases and exceptions
   - Market condition filters
   - Recovery/safety protocols
   - Any other special handling

5. WORKFLOW SECTION
   - Step-by-step flow of how a trade is executed
   - Decision points with conditions
   - What happens at each stage

6. EXAMPLES SECTION
   - 3-5 detailed trade walkthroughs
   - Show real scenarios from my backtest
   - Show both winning and losing trades

Please create this specification now. Be thorough and explicit.
```

### Step 3: Review and Refine

The AI will create LOGIC_SPECIFICATION.md

Your job: Review it carefully
- [ ] Does it match your strategy exactly?
- [ ] Are all numbers correct?
- [ ] Are all conditions clear?
- [ ] Are examples accurate?

If something is wrong:
```
I found an issue:

In the ENTRY LOGIC section, you say "[incorrect description]"
But actually, my rule is "[correct description]"

Please fix this.
```

Let the AI fix issues until the specification is perfect.

---

## 🎯 WHAT THE AI CAN DO FREELY

**Don't restrict the AI.** Let it:
- Ask clarification questions
- Point out gaps or inconsistencies
- Suggest alternative explanations
- Request backtest data to verify logic
- Ask for additional details
- Challenge unclear conditions
- Propose edge cases you might not have considered

The AI will create a MUCH better specification if you let it think freely.

---

## ✅ PHASE 2 CHECKLIST

Before Phase 2:
- [ ] Phase 1 is 100% complete
- [ ] PHASE_1_APPROVED.txt exists
- [ ] DeepSeek AI is running
- [ ] You're ready to review specification

During Phase 2:
- [ ] AI asks questions about your strategy
- [ ] You answer any questions that come up
- [ ] AI creates LOGIC_SPECIFICATION.md

After Phase 2:
- [ ] LOGIC_SPECIFICATION.md exists
- [ ] You've reviewed it completely
- [ ] Everything is correct and detailed
- [ ] Ready for Phase 3

---

## 📊 WHAT PHASE 2 OUTPUT LOOKS LIKE

**LOGIC_SPECIFICATION.md** will contain:

```
LOGIC SPECIFICATION
===================

STRATEGY: [Your strategy name]
SYMBOL: [Trading pair]
TIMEFRAME: [1H, 4H, M15, etc.]

=====================================
SECTION 1: ENTRY LOGIC
=====================================

Entry signals are generated when ALL of these conditions are true:

Condition 1: Time Filter
   Description: [Exact description]
   Rule: [If/Then format]
   Range: [Specific times]
   Example: [Real example from backtest]

Condition 2: Indicator 1
   Description: [Exact description]
   Formula: [Math if applicable]
   Threshold: [Exact number]
   Example: [Real example]

[Continue for all entry conditions...]

Entry Example (Real Trade from Backtest):
   Date: 2025-01-05
   Time: 08:30
   Condition 1 met: YES (Time is 08:30)
   Condition 2 met: YES (RSI = 72, > 70)
   Condition 3 met: YES (Price = 1.0960, MA = 1.0920)
   Result: ENTRY SIGNAL GENERATED - BUY

=====================================
SECTION 2: EXIT LOGIC
=====================================

Take Profit Rule:
   Condition: [When to close for profit]
   Calculation: [How TP is calculated]
   Example: [Real example with numbers]

Stop Loss Rule:
   Condition: [When to close for loss]
   Calculation: [How SL is calculated]
   Example: [Real example with numbers]

Trailing Stop (if applicable):
   Trigger: [When trailing starts]
   Trail Amount: [How much to trail]
   Example: [Real scenario]

Time Exit (if applicable):
   Condition: [When to close based on time]
   Example: [Real example]

=====================================
SECTION 3: RISK MANAGEMENT
=====================================

Lot Size Calculation:
   Formula: [Exact math]
   Account: [Account size assumed]
   Risk per trade: [Amount or percentage]
   Example: Account $10000, Risk $100 → Lot = 0.01

Position Limits:
   Maximum open positions: [Number]
   Max daily loss: [Amount or percentage]
   Max weekly loss: [If applicable]

Examples:
   Scenario 1: Already have 3 positions, new signal arrives
               Result: SKIP (already at max)
   
   Scenario 2: Daily loss already $500, new trade would risk $100
               Result: ALLOWED (total would be $600, within limit)

=====================================
SECTION 4: SPECIAL CONDITIONS
=====================================

[All edge cases and special rules documented]

=====================================
SECTION 5: COMPLETE WORKFLOW
=====================================

OnTick() Flow:
   Step 1: Check time filter (Is it trading time?)
   Step 2: Check entry conditions (Is signal present?)
   Step 3: Check position limits (Can we open more?)
   Step 4: Calculate lot size
   Step 5: Open trade
   Step 6: Check exit conditions (Should we close?)
   Step 7: Close if conditions met

=====================================
SECTION 6: EXAMPLES
=====================================

[3-5 detailed trade walkthroughs with all calculations]

```

---

## 💡 WHAT TO EXPECT

The AI will:
- ✅ Ask clarifying questions
- ✅ Request backtest examples
- ✅ Point out unclear areas
- ✅ Suggest improvements
- ✅ Create comprehensive documentation
- ✅ Show its understanding through examples

Let it do all this. Don't restrict it.

---

## 🔄 PROGRESS UPDATE

When Phase 2 is complete, create:

**File: PROGRESS_UPDATE.txt**

Add this line:
```
PHASE 2 COMPLETE
================

Date Completed: [Today's date]
Duration: [How many hours]
AI Used: DeepSeek V3 (Key #1)

WHAT WAS ACCOMPLISHED:
- AI read my Phase 1 approval
- Asked clarifying questions
- Created detailed specification
- LOGIC_SPECIFICATION.md finalized

STATUS: Ready for Phase 3
NEXT: Switch to Gemini (Key #2) and start Phase 3
```

---

**NEXT STEP: Phase 2 is now running. Review the LOGIC_SPECIFICATION.md and confirm it's correct!**

Good luck! 🚀
