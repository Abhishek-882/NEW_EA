# 📋 PHASES 3-7 DETAILED INSTRUCTIONS

**Complete user-focused guide for remaining phases**

---

## 🎯 PHASE 3: CODE ARCHITECTURE PLANNING

**AI:** Gemini 1.5 Pro (Key #2)  
**Duration:** 1-2 hours  
**Input:** LOGIC_SPECIFICATION.md  
**Output:** CODE_ARCHITECTURE.md, CLASS_DESIGNS.md, DATA_FLOW.md  

### What to Tell the AI:

```
Read LOGIC_SPECIFICATION.md

Now you are Phase 3. Your job is to design the MQL5 code architecture.

DO NOT write actual code yet - just plan it.

I need you to design:

1. CODE_ARCHITECTURE.md
   - What files are needed (.mqh and .mq5)
   - How many lines each file should be
   - Dependencies between files
   - Compilation order
   - File purposes

2. CLASS_DESIGNS.md
   For each class:
   - Class name and purpose
   - What public methods it has (what it can do)
   - What private methods it has (internal operations)
   - What variables it stores
   - How it implements the logic from LOGIC_SPECIFICATION.md

3. DATA_FLOW.md
   - How does OnTick() work?
   - What data flows between classes?
   - What are the decision points?
   - How is a trade executed step by step?

The design must:
- Implement 100% of LOGIC_SPECIFICATION.md
- Be clean and modular
- Have good separation of concerns
- Be ready for coding in Phase 4

Please create these 3 documents now. Be thorough.
```

### After Phase 3:

You'll have architectural designs. Review them:
- [ ] Does it match your strategy?
- [ ] Are all files clearly defined?
- [ ] Do classes make sense?
- [ ] Is the data flow clear?

If issues, ask AI to revise.

### Progress Update:

```
PHASE 3 COMPLETE
================
Date: [Today]
AI Used: Gemini 1.5 Pro (Key #2)
Output: CODE_ARCHITECTURE.md created ✓
Next: Phase 4 - Code Generation
```

---

## 🎯 PHASE 4: CODE GENERATION

**AI:** DeepSeek V3 (Key #2)  
**Duration:** 4-6 hours  
**Input:** CODE_ARCHITECTURE.md, LOGIC_SPECIFICATION.md  
**Output:** 7 MQL5 files (.mq5 + 6 .mqh)  

### What to Tell the AI:

```
Read CODE_ARCHITECTURE.md and LOGIC_SPECIFICATION.md

Now you are Phase 4. Your job is to generate actual MQL5 code.

Generate code file by file, in this order:

1. Config.mqh (~500 lines) - Parameters and constants
2. IndicatorEngine.mqh (~2000 lines) - Calculate indicators
3. SignalProvider.mqh (~2000 lines) - Generate entry/exit signals
4. RiskManager.mqh (~1000 lines) - Calculate lot sizes, limits
5. TradeExecutor.mqh (~2500 lines) - Execute trades
6. StateManager.mqh (~1500 lines) - Track positions
7. MainEA.mq5 (~1500 lines) - Main EA orchestration

For EACH file:

1. Generate the code
2. Include all error handling
3. Add detailed comments
4. Make it compilable
5. Implement 100% of the architecture

After generating each file, I will:
1. Copy it to MetaEditor
2. Press F7 to compile
3. Report compilation result:
   - If ERROR: Show error + problematic code to you
   - If SUCCESS: Move to next file

DO NOT proceed to next file until current file compiles with 0 errors.

Start with Config.mqh now.
```

### What to Do in Phase 4:

1. **Receive code:** AI sends file code
2. **Copy to MetaEditor:** Select all code, paste to MetaEditor
3. **Compile:** Press F7
4. **If error:**
   ```
   Compilation error in [FILENAME]:
   ERROR: [Copy exact error from MetaEditor]
   
   Here's the problematic code:
   [Copy code section that failed]
   ```
   Then AI fixes it

5. **If success:** Move to next file

### Files Needed After Phase 4:
- [ ] Config.mqh (compiles ✓)
- [ ] IndicatorEngine.mqh (compiles ✓)
- [ ] SignalProvider.mqh (compiles ✓)
- [ ] RiskManager.mqh (compiles ✓)
- [ ] TradeExecutor.mqh (compiles ✓)
- [ ] StateManager.mqh (compiles ✓)
- [ ] MainEA.mq5 (compiles ✓)

### Progress Update:

```
PHASE 4 COMPLETE
================
Date: [Today]
Duration: [Hours spent]
AI Used: DeepSeek V3 (Key #2)

ALL 7 FILES GENERATED AND COMPILED:
✓ Config.mqh
✓ IndicatorEngine.mqh
✓ SignalProvider.mqh
✓ RiskManager.mqh
✓ TradeExecutor.mqh
✓ StateManager.mqh
✓ MainEA.mq5

Next: Phase 5 - Verification
```

---

## 🎯 PHASE 5: LOGIC VERIFICATION

**AI:** Gemini 1.5 Pro (Key #3)  
**Duration:** 2-3 hours  
**Input:** LOGIC_SPECIFICATION.md + all code files  
**Output:** VERIFICATION_REPORT.md  

### What to Tell the AI:

```
Read LOGIC_SPECIFICATION.md and all 7 code files.

Now you are Phase 5. Your job is to verify that the code 
I built in Phase 4 actually matches my original logic specification.

Compare the specification to the code line-by-line.

For EACH rule in the specification, check:
- Is this implemented in the code? (YES/NO/PARTIALLY)
- Is it implemented correctly? (YES/NO)
- Are the exact values used? (YES/NO)
- Is there any mismatch? (YES/NO)

Create VERIFICATION_REPORT.md that shows:

For each logic section:

✅ MATCHES (100% correct)
   Spec: [Location in specification]
   Code: [Location in code]
   Details: [What matches exactly]

⚠️ PARTIAL (Partially correct)
   Spec: [Location in specification]
   Code: [Location in code]
   Issue: [What's not quite right]
   Impact: [What needs fixing]

❌ MISMATCH (Wrong or missing)
   Spec: [Location in specification]
   Code: [Location in code or "NOT FOUND"]
   Issue: [What's wrong]
   Required: [What should be there instead]

Create a comprehensive verification report.
```

### What to Do in Phase 5:

1. Review the VERIFICATION_REPORT.md completely
2. Check if any issues are listed
3. Note down the ❌ MISMATCH items (those need fixing)
4. Note down the ⚠️ PARTIAL items (those may need fixing)

### Expected Output:

```
VERIFICATION_REPORT.md contains:

✅ ENTRY: Time Filter - MATCHES
✅ ENTRY: RSI Condition - MATCHES
✅ ENTRY: MA Filter - MATCHES
❌ ENTRY: Daily Loss Limit - MISMATCH (NOT FOUND IN CODE)
⚠️ RISK: Lot calculation - PARTIAL (Uses formula but missing edge case)
✅ EXIT: Take Profit - MATCHES
...

SUMMARY:
- Total checks: 47
- Matches: 43 ✅
- Partial: 2 ⚠️
- Mismatches: 2 ❌

ISSUES TO FIX IN PHASE 6:
1. Daily Loss Limit not implemented
2. Lot calculation missing edge case
```

### Progress Update:

```
PHASE 5 COMPLETE
================
Date: [Today]
AI Used: Gemini 1.5 Pro (Key #3)

VERIFICATION_REPORT.md created:
- 43 items match correctly ✅
- 2 items partially correct ⚠️
- 2 items need fixing ❌

Next: Phase 6 - Error Fixing
```

---

## 🎯 PHASE 6: ERROR FIXING

**AI:** DeepSeek V3 (Key #3)  
**Duration:** 2-3 hours  
**Input:** VERIFICATION_REPORT.md + code files  
**Output:** Fixed code files  

### What to Tell the AI:

```
Read VERIFICATION_REPORT.md

I've run verification and found these issues that need fixing:

[Copy the ❌ MISMATCH items from report]
[Copy the ⚠️ PARTIAL items from report]

For each issue:
1. Understand what should be fixed
2. Show me the corrected code
3. Explain what you changed
4. I'll compile to verify

Then move to the next issue.

After all fixes are done, we'll re-run verification 
to confirm everything matches 100%.

Start with Issue #1.
```

### What to Do in Phase 6:

For EACH issue from verification report:

1. **Receive fixed code** from AI
2. **Copy to MetaEditor** - Replace the problematic section
3. **Compile** - Press F7
4. **Report result:**
   ```
   Issue #1 Fix:
   Compilation: SUCCESS ✓
   [OR]
   Compilation: ERROR
   Error message: [Copy exact error]
   ```
5. **Move to next issue**

After all fixes:
```
Tell AI: "Re-run Phase 5 verification now to confirm all fixes work"

AI will verify again and should report:
✅ ALL CHECKS PASS (0 mismatches)
```

### Progress Update:

```
PHASE 6 COMPLETE
================
Date: [Today]
AI Used: DeepSeek V3 (Key #3)

FIXES APPLIED:
✓ Issue #1: Daily Loss Limit - Fixed
✓ Issue #2: Lot Calculation - Fixed

RE-VERIFICATION: All checks pass ✓

Next: Phase 7 - Final Validation
```

---

## 🎯 PHASE 7: FINAL VALIDATION & PACKAGING

**AI:** Gemini 1.5 Pro (Key #4)  
**Duration:** 1-2 hours  
**Input:** All fixed code files  
**Output:** Complete EA package ready for download  

### What to Tell the AI:

```
Read all the fixed code files from Phase 6.

Now you are Phase 7 - the final phase.

Your job is to:

1. FINAL CHECKS
   - Do all files compile with 0 errors? YES/NO
   - Do all files compile with 0 warnings? YES/NO
   - Does MainEA.mq5 compile? YES/NO
   - Are all #include statements working? YES/NO

2. FINAL LOGIC VERIFICATION
   - Does code 100% match LOGIC_SPECIFICATION.md? YES/NO
   - Are all entry rules implemented? YES/NO
   - Are all exit rules implemented? YES/NO
   - Are all risk rules implemented? YES/NO
   - Are all Phase 6 fixes applied? YES/NO

3. CREATE DOCUMENTATION
   - README_EA.md (what this EA does)
   - INSTALLATION.md (how to install in MetaTrader)
   - PARAMETERS.md (all input parameters)
   - TROUBLESHOOTING.md (common issues and fixes)

4. ORGANIZE PACKAGE
   Create folder structure:
   
   📁 MQL5_EA_COMPLETE/
   ├─ 📁 Source Code/
   │  ├─ MainEA.mq5
   │  ├─ Config.mqh
   │  ├─ IndicatorEngine.mqh
   │  ├─ SignalProvider.mqh
   │  ├─ RiskManager.mqh
   │  ├─ TradeExecutor.mqh
   │  └─ StateManager.mqh
   ├─ 📁 Documentation/
   │  ├─ README_EA.md
   │  ├─ INSTALLATION.md
   │  ├─ PARAMETERS.md
   │  └─ TROUBLESHOOTING.md
   ├─ 📁 Reference/
   │  ├─ LOGIC_SPECIFICATION.md
   │  ├─ CODE_ARCHITECTURE.md
   │  └─ VERIFICATION_REPORT.md
   └─ 📁 Build Logs/
      └─ PHASE_LOGS.txt

5. CREATE FINAL_PACKAGE.md
   Document that summarizes:
   - Project name and purpose
   - What the EA does
   - How to install
   - How to use
   - Testing checklist
   - Support information

If all checks pass, declare project COMPLETE ✓
```

### What to Do in Phase 7:

1. Review all checks pass
2. Download all documentation files
3. Review README, INSTALLATION, and PARAMETERS files
4. Confirm everything is correct
5. When satisfied, declare Phase 7 complete

### Final Deliverables:

You receive:
- ✅ MainEA.mq5 (ready to use)
- ✅ 6 .mqh files (support files)
- ✅ Complete documentation
- ✅ README file
- ✅ Installation guide
- ✅ Parameter definitions
- ✅ Troubleshooting guide
- ✅ Project summary

### Progress Update:

```
PHASE 7 COMPLETE - PROJECT FINISHED! 🎉
====================================

Date: [Today]
AI Used: Gemini 1.5 Pro (Key #4)

FINAL CHECKS: All passed ✓
- Compilation: 0 errors, 0 warnings ✓
- Logic: 100% match ✓
- Documentation: Complete ✓
- Package: Organized ✓

STATUS: PRODUCTION-READY EA COMPLETE ✓

DELIVERABLES:
✓ MainEA.mq5 (~1,500 lines)
✓ 6 support files (~10,500 lines)
✓ Complete documentation (~50 pages)
✓ Installation guide
✓ Parameter documentation
✓ Troubleshooting guide

TOTAL: ~12,000 lines of production-ready MQL5 code

NEXT STEP: Download EA package and deploy to MetaTrader 5!
```

---

## 🎯 MASTER AI UPDATE TEMPLATE

Use this to update the master AI after EACH phase:

```
MASTER AI - PROGRESS UPDATE
============================

Project: [Your EA project name]
Current Phase: [1-7]
Status: COMPLETE ✓

PHASE [N] RESULTS:
==================

Date Completed: [Date]
Time Spent: [Hours]
AI Model Used: [Gemini/DeepSeek]
API Key Used: [#1, #2, #3, #4]

DELIVERABLES CREATED:
- [File names created]
- [Quality metrics]
- [Any issues encountered]

NEXT PHASE:
- Switch to: [Gemini/DeepSeek]
- API Key: [#X]
- Duration: [Estimated hours]

NOTES:
[Any special notes or issues]

STATUS: Ready for Phase [N+1]
```

---

## ✅ COMPLETE PHASE CHECKLIST

Phase 1:
- [ ] Strategy described completely
- [ ] AI asked clarifying questions
- [ ] PHASE_1_APPROVED.txt created
- [ ] Ready for Phase 2

Phase 2:
- [ ] LOGIC_SPECIFICATION.md created
- [ ] Reviewed and correct
- [ ] Ready for Phase 3

Phase 3:
- [ ] CODE_ARCHITECTURE.md created
- [ ] CLASS_DESIGNS.md created
- [ ] DATA_FLOW.md created
- [ ] Ready for Phase 4

Phase 4:
- [ ] All 7 files generated
- [ ] All compile with 0 errors
- [ ] Ready for Phase 5

Phase 5:
- [ ] VERIFICATION_REPORT.md created
- [ ] Issues identified
- [ ] Ready for Phase 6

Phase 6:
- [ ] All issues fixed
- [ ] All files re-compile
- [ ] Ready for Phase 7

Phase 7:
- [ ] Final checks pass
- [ ] Documentation complete
- [ ] Package organized
- [ ] EA COMPLETE! 🎉

---

**REMEMBER: Let the AI think freely. Don't restrict it with strict instructions.**

**The AI knows how to build EAs. Trust the process!**

Good luck! 🚀
