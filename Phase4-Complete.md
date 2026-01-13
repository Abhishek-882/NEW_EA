═══════════════════════════════════════════════════════════════════════════════
PHASE 4: COMPLETE EA IMPLEMENTATION GUIDE
Multi-Symbol, Multi-Timeframe Mean Reversion Trading Strategy (v2.0)
═══════════════════════════════════════════════════════════════════════════════

Audience: Development Team, Implementation Engineers
Input: Phase 3 Architecture (v2.0) + Explained Specification (v2.0)
Output: Production-Ready MQL5 Code with Testing & Documentation
Status: READY FOR IMPLEMENTATION

═══════════════════════════════════════════════════════════════════════════════
PHASE 4 OVERVIEW & CORE PRINCIPLE
═══════════════════════════════════════════════════════════════════════════════

KEY PRINCIPLE: Use Phase 3 Architecture as BLUEPRINT, Not Template

❌ Don't blindly follow a generic checklist
✅ DO understand YOUR architecture deeply before coding
✅ DO customize implementation for YOUR specific strategy
✅ DO challenge architecture when implementation reveals issues
✅ DO think ahead: What will Phase 5 need to test?

What This Phase Accomplishes:
  Input:  ✓ Phase 3 architecture (10 modules defined)
          ✓ Phase 3 specifications (data flows, state machines)
          ✓ Locked strategy (Explained v2.0)
          
  Output: ✓ All 10 modules fully implemented in MQL5
          ✓ All TODOs replaced with actual code
          ✓ Unit tests for each module (custom, not generic)
          ✓ Integration tests (multi-symbol, multi-timeframe flows)
          ✓ System tests (full EA behavior)
          ✓ Code review passed
          ✓ Complete documentation
          ✓ Ready for Phase 5 backtesting

═══════════════════════════════════════════════════════════════════════════════
STAGE 1: DEVELOPMENT SETUP & ARCHITECTURE REVIEW (1-2 Days)
═══════════════════════════════════════════════════════════════════════════════

1.1 ARCHITECTURE DEEP DIVE CHECKLIST
─────────────────────────────────────────────────────────────────────────────

Before writing a SINGLE line of code, answer these 8 questions about YOUR architecture:

[YOUR ARCHITECTURE UNDERSTANDING CHECKLIST]

☐ How many modules? 
   Answer: 10 modules (DataProvider, PatternRecognizer, NewsFilter, EntryManager,
           ExitCalculator, TradeManager, StateMachine, SymbolOrchestrator,
           MultiTimeframeAnalyzer, Logger)

☐ What does each module do?
   DataProvider: Fetch OHLCV, calculate BB/ADX/Volume MA per symbol/timeframe
   PatternRecognizer: Detect 3-candle band-touch patterns (SELL/BUY)
   NewsFilter: Check calendar, block entries ±60min around events
   EntryManager: Place SELL/BUY orders with static SL/TP (simplified v2.0)
   ExitCalculator: Monitor static TP (opposite band) + SL (entry band)
   TradeManager: Execute exits, record closed trades
   StateMachine: Orchestrate ONE symbol's state (7-state machine)
   SymbolOrchestrator: Manage array of StateMachines (one per symbol)
   MultiTimeframeAnalyzer: Optional confirmation from higher timeframe
   Logger: Log trades, entries, exits, blocks

☐ How do modules communicate?
   SymbolOrchestrator.OnBar(symbol)
     → Calls StateMachine.OnCandle(primary_data, confirm_data, news_filter)
       → Uses DataProvider.GetCurrentData()
       → Uses PatternRecognizer.CheckPatterns()
       → Uses NewsFilter.Check()
       → Uses MultiTimeframeAnalyzer.Confirm*()
       → Calls EntryManager.Enter*() if entry conditions met
       → Calls ExitCalculator.CheckExits() if trade open
       → Calls TradeManager.ClosePosition() if exit triggered
       → Calls Logger.LogEvent()

☐ What's the execution sequence?
   Per symbol, per candle:
   1. Fetch price data (primary TF + optional confirm TF)
   2. Check news filter (block entries if in window)
   3. Check multi-timeframe confirmation (if enabled)
   4. Recognize 3-candle patterns (1st, 2nd, 3rd validation)
   5. Check exits if position open (TP/SL comparisons)
   6. Transition state machine
   7. Log any changes

☐ Where are the bottlenecks?
   Performance Critical:
   • DataProvider.GetCurrentData(): Called per symbol per candle
     → Must cache indicator calculations (don't recalculate every time)
   • PatternRecognizer.CheckPatterns(): Called per symbol per candle
     → Must track state efficiently (minimal looping)
   • NewsFilter.Check(): Called per symbol per candle
     → Pre-load calendar (don't search every time)
   • Order execution: Must be < 1000ms
   
   Test with: 3 symbols × 5 timeframes = 15 analysis loops per candle

☐ Where will errors likely occur?
   High-Risk Areas (Test These):
   • Bollinger Band calculations when history < 20 bars
   • ADX calculations when ADX value is exactly 25 (boundary)
   • Volume division by zero (if AvgVol is 0)
   • News filter with missing calendar data
   • Multi-symbol conflicts (same symbol listed twice)
   • Multi-timeframe: confirm TF < primary TF (invalid)
   • State machine: Pattern timeout (stays in DETECTED too long)
   • Entry execution: Order rejection from broker
   • Position closing: Multiple close attempts on same position

☐ How to test each module?
   DataProvider:
     • Test with real EURUSD data (verify BB matches MT5)
     • Test with missing data (< 20 bars)
     • Test BB convergence/divergence
     • Test ADX boundary values
   
   PatternRecognizer:
     • Feed mock OHLC data with predefined patterns
     • Test band-touch detection
     • Test retreat validation
     • Test ADX threshold application
     • Test volume multiplier
   
   NewsFilter:
     • Test with event inside window
     • Test with event outside window
     • Test time boundary (exactly at event start)
     • Test multiple events in same window
   
   EntryManager:
     • Test order creation (don't actually place orders)
     • Test SL/TP calculation (opposite/same-side bands)
     • Test lot size application
   
   ExitCalculator:
     • Feed positions with known TP/SL levels
     • Test TP hit detection (SELL: close ≤ TP)
     • Test SL hit detection (SELL: close ≥ SL)
     • Test BUY logic (opposite conditions)
     • Test both hit same bar (TP priority)
   
   TradeManager:
     • Test position tracking
     • Test PnL calculation
     • Test position close
   
   StateMachine:
     • Test all 7-state transitions
     • Test multi-timeframe confirmation flow
     • Test news filter blocking at 3 points
     • Test pattern timeout (don't hang)
   
   SymbolOrchestrator:
     • Test with 1 symbol (simple case)
     • Test with 3 symbols (multi-symbol flow)
     • Test position limit (max 1 per symbol)
     • Test no interference between symbols

☐ What are extension points?
   Future enhancements should integrate here:
   
   Dynamic Position Sizing:
     • Point: EntryManager.EnterSell/EnterBuy signature
     • Add parameter: risk_percent (calculate from account equity)
   
   Partial Take Profits:
     • Point: ExitCalculator returns array of exits
     • Current: Single TP level → Future: Multiple TP levels with sizes
   
   Daily Loss Limits:
     • Point: SymbolOrchestrator.ProcessAllSymbols()
     • Add: RiskManager checking daily P&L
   
   Correlation Filters:
     • Point: SymbolOrchestrator.AddSymbol()
     • Add: Check correlation with open positions
   
   ML Signal Enhancement:
     • Point: StateMachine.OnCandle() after pattern detected
     • Add: MLPredictor.GetConfidence() to validate entry

If you CAN'T answer all 8 → Stop, re-read Phase 3 architecture, clarify
If you CAN answer all 8 → Ready to proceed

═══════════════════════════════════════════════════════════════════════════════

1.2 DEPENDENCY-BASED IMPLEMENTATION ORDER
─────────────────────────────────────────────────────────────────────────────

CRITICAL: Implement modules in dependency order, not arbitrary order

[IMPLEMENTATION DEPENDENCY MAP]

Layer 0 (No dependencies):
  └─ DataStructures.mqh (all struct definitions)
  └─ NewsCalendarData.mqh (embedded event data)

Layer 1 (Depends only on Layer 0):
  └─ DataProvider (fetches data, calculates indicators)
  └─ NewsFilter (checks calendar against current time)
  └─ Logger (logs to file/comment)

Layer 2 (Depends on Layer 1):
  └─ PatternRecognizer (uses DataProvider output)
  └─ ExitCalculator (logic independent, uses DataStructures)

Layer 3 (Depends on Layer 1-2):
  └─ EntryManager (uses DataProvider for band levels)
  └─ MultiTimeframeAnalyzer (uses DataProvider for confirm TF)

Layer 4 (Depends on Layer 1-3):
  └─ TradeManager (manages positions, uses Logger)

Layer 5 (Depends on Layer 1-4):
  └─ StateMachine (orchestrates all per-symbol logic)

Layer 6 (Depends on all):
  └─ SymbolOrchestrator (manages array of StateMachines)

[CONCRETE IMPLEMENTATION SEQUENCE]

Week 1:
  Day 1: DataStructures.mqh + NewsCalendarData.mqh
  Day 2: DataProvider (unit tests with real data)
  Day 3: NewsFilter + Logger (unit tests)
  Day 4: PatternRecognizer (unit tests with mock data)
  Day 5: ExitCalculator (unit tests)

Week 2:
  Day 1: EntryManager (unit tests, no actual orders yet)
  Day 2: MultiTimeframeAnalyzer (unit tests)
  Day 3: TradeManager (unit tests)
  Day 4: StateMachine (unit tests for state transitions)
  Day 5: SymbolOrchestrator (unit tests with single symbol first)

Week 3:
  Day 1-3: Integration testing (modules together)
  Day 4-5: Multi-symbol testing, multi-timeframe testing

Week 4:
  Day 1-2: System testing (full EA behavior)
  Day 3-4: Code review + fixes
  Day 5: Documentation

Why This Order?
  ✓ Can test DataProvider independently (no dependencies)
  ✓ Can test PatternRecognizer with mock data (doesn't need real market data)
  ✓ Can test StateMachine state transitions before SymbolOrchestrator
  ✓ Can test SymbolOrchestrator with 1 symbol before scaling
  ✓ Catch problems early (layers built on correct foundation)

═══════════════════════════════════════════════════════════════════════════════

1.3 DEVELOPMENT BEST PRACTICES
─────────────────────────────────────────────────────────────────────────────

[DEVELOPMENT CHECKLIST]

Version Control:
  ☐ Git initialized (or similar VCS)
  ☐ .gitignore configured (ignore .exe, backtest results, etc)
  ☐ Initial commit: "Phase 4: Initialize repository"
  ☐ Branches for experimental features (not on main)
  ☐ Meaningful commit messages per completed module

Code Organization:
  ☐ One module per .mqh file (or logically grouped)
  ☐ File names match module names (DataProvider.mqh, etc)
  ☐ Main EA file: EUR_NZD_MultiSymbol_EA.mq5
  ☐ Utilities in separate file: Utilities.mqh
  ☐ All includes at top of main file

Testing Strategy:
  ☐ Unit tests written AS you code (not after)
  ☐ Test function signature: void Test_ModuleName_MethodName()
  ☐ Each method has 2-3 test cases (normal, edge, error)
  ☐ Tests can be disabled in production (compile flag)
  ☐ All tests documented (what they test, why)

Documentation:
  ☐ Method headers: Purpose, inputs, outputs, error codes
  ☐ Complex logic: Comments explaining WHY (not just what)
  ☐ Design decisions: Document non-obvious choices
  ☐ Error codes: Define all error returns (success, fail, etc)
  ☐ Create README: Build instructions, test running, deployment

Code Quality:
  ☐ Variable names clear (not single letters except loops)
  ☐ Constants for magic numbers (PERIOD_BB = 20, THRESHOLD_ADX = 25)
  ☐ Consistent indentation (2 or 4 spaces)
  ☐ Line length reasonable (< 100 characters)
  ☐ No dead code (delete unused methods)

Robustness:
  ☐ All error cases handled (return error code, don't crash)
  ☐ Input validation on all external data
  ☐ Graceful degradation (eg: skip entry if data unavailable)
  ☐ No hardcoded values (use constants)
  ☐ No assumptions about data validity

Code Review:
  ☐ Self-review before marking complete
  ☐ Peer review if possible (another dev)
  ☐ Security review: No credentials, no SQL injection, no buffer overflows
  ☐ Performance review: No unnecessary loops, efficient caching

User Action at Stage 1:
  ☐ APPROVE - Ready to implement
  ☐ QUESTION - Need clarification on architecture
  ☐ CONCERN - Worried about specific implementation challenge

═══════════════════════════════════════════════════════════════════════════════
STAGE 2: MODULE IMPLEMENTATION
═══════════════════════════════════════════════════════════════════════════════

2.1 MODULE IMPLEMENTATION FRAMEWORK
─────────────────────────────────────────────────────────────────────────────

For EACH module, follow this pattern (customize for YOUR specific logic):

[IMPLEMENTATION FRAMEWORK]

STEP 1: Understand Module Purpose
  └─ Read Phase 3 module definition
    ├─ What inputs?
    ├─ What outputs?
    ├─ What state maintained?
    ├─ What responsibilities?
    └─ Write this understanding as comments

STEP 2: Define Data Structures (if new)
  └─ Create struct for module's data
    ├─ Input struct (what caller provides)
    ├─ Output struct (what module returns)
    └─ Internal state struct (if needed)

STEP 3: Implement Core Logic
  └─ Write the actual algorithm
    ├─ Based on YOUR specification (Explained v2.0)
    ├─ Efficient implementation (no unnecessary loops)
    ├─ Clear variable names
    ├─ Comments for complex parts
    └─ Handle all input variations

STEP 4: Add Error Handling
  └─ What can go wrong in THIS module?
    ├─ Invalid inputs → validate, return error
    ├─ Calculation errors → fallback gracefully
    ├─ State issues → reset or recover
    └─ Never crash → always return error code

STEP 5: Write Unit Tests
  └─ Test THIS module independently
    ├─ Normal cases (expected inputs)
    ├─ Edge cases (boundary values)
    ├─ Error cases (invalid inputs)
    ├─ All tests pass before moving on
    └─ Tests documented

STEP 6: Integrate with Previous Modules
  └─ Connect THIS module to prior ones
    ├─ Call previous module methods
    ├─ Pass data in expected format
    ├─ Verify data flows correctly
    ├─ Integration test with real data
    └─ Performance acceptable

STEP 7: Document THIS Module
  └─ Create module documentation
    ├─ Purpose (plain English)
    ├─ How to use methods
    ├─ Error codes it returns
    ├─ How to test it
    ├─ Design decisions made
    └─ Any known limitations

This is NOT a rigid 7-step process
This is a FRAMEWORK you adapt for each module

═══════════════════════════════════════════════════════════════════════════════

2.2 DETAILED IMPLEMENTATION EXAMPLES
─────────────────────────────────────────────────────────────────────────────

EXAMPLE 1: DataProvider Module (Foundation Layer)

[DataProvider.mqh - SIMPLIFIED EXAMPLE]

```cpp
// DataProvider.mqh
// Purpose: Fetch OHLCV and calculate BB/ADX/Volume MA
// For: Multi-symbol, multi-timeframe analysis
// Created: [DATE]

#ifndef DATAPROVIDER_H
#define DATAPROVIDER_H

#include "DataStructures.mqh"

class DataProvider {
private:
  string m_symbol;
  int m_timeframe;
  int m_handle_adx;  // ADX indicator handle
  
  // Caching (efficiency)
  PriceData m_cached_data;
  datetime m_cached_time;
  bool m_cache_valid;
  
  // Constants (from Explained v2.0)
  const int BB_PERIOD = 20;
  const double BB_DEVIATION = 2.0;
  const int ADX_PERIOD = 14;
  const int VOLUME_MA_PERIOD = 20;
  
public:
  // Constructor
  DataProvider(string symbol, int timeframe) {
    m_symbol = symbol;
    m_timeframe = timeframe;
    m_cache_valid = false;
    
    // Create ADX indicator handle (once)
    m_handle_adx = iADX(m_symbol, m_timeframe, ADX_PERIOD);
    if (m_handle_adx == INVALID_HANDLE) {
      // ERROR: Handle creation failed
      // Log error, but don't crash
    }
  }
  
  // Destructor
  ~DataProvider() {
    if (m_handle_adx != INVALID_HANDLE) {
      IndicatorRelease(m_handle_adx);
    }
  }
  
  // Main method: Get current OHLCV + calculated indicators
  PriceData GetCurrentData() {
    // Check cache (if same candle, return cached)
    datetime bar_time = iTime(m_symbol, m_timeframe, 0);
    if (m_cache_valid && m_cached_time == bar_time) {
      return m_cached_data;  // Cache hit
    }
    
    // CACHE MISS: Fetch new data
    PriceData data = FetchOHLCV();
    if (!CalculateBollingerBands(data)) {
      // ERROR: BB calculation failed
      // Return error indicator in data struct
      data.error = true;
      return data;
    }
    
    if (!CalculateADX(data)) {
      // ERROR: ADX calculation failed
      data.error = true;
      return data;
    }
    
    if (!CalculateVolume(data)) {
      // ERROR: Volume calculation failed
      data.error = true;
      return data;
    }
    
    // Cache this data
    m_cached_data = data;
    m_cached_time = bar_time;
    m_cache_valid = true;
    
    return data;
  }
  
private:
  // Fetch OHLCV
  PriceData FetchOHLCV() {
    PriceData data = {};
    
    // Copy current candle data
    if (CopyOpen(m_symbol, m_timeframe, 0, 1, &data.open) != 1) {
      // ERROR handling
      return data;
    }
    if (CopyHigh(m_symbol, m_timeframe, 0, 1, &data.high) != 1) {
      return data;
    }
    if (CopyLow(m_symbol, m_timeframe, 0, 1, &data.low) != 1) {
      return data;
    }
    if (CopyClose(m_symbol, m_timeframe, 0, 1, &data.close) != 1) {
      return data;
    }
    if (CopyTickVolume(m_symbol, m_timeframe, 0, 1, &data.volume) != 1) {
      return data;
    }
    
    return data;
  }
  
  // Calculate Bollinger Bands
  bool CalculateBollingerBands(PriceData& data) {
    // Need 20 closes for SMA
    double closes[20];
    if (CopyClose(m_symbol, m_timeframe, 0, 20, closes) != 20) {
      // ERROR: Not enough history
      return false;
    }
    
    // SMA-20
    double sma = 0;
    for (int i = 0; i < 20; i++) {
      sma += closes[i];
    }
    sma /= 20;
    data.sma_20 = sma;
    
    // Standard Deviation
    double sum_sq_diff = 0;
    for (int i = 0; i < 20; i++) {
      double diff = closes[i] - sma;
      sum_sq_diff += diff * diff;
    }
    double std_dev = MathSqrt(sum_sq_diff / 20);
    
    // Bands
    data.bb_middle = sma;
    data.bb_upper = sma + (2.0 * std_dev);
    data.bb_lower = sma - (2.0 * std_dev);
    
    return true;
  }
  
  // Calculate ADX
  bool CalculateADX(PriceData& data) {
    if (m_handle_adx == INVALID_HANDLE) {
      // ERROR: Handle invalid
      return false;
    }
    
    double adx[1];
    if (CopyBuffer(m_handle_adx, 0, 0, 1, adx) != 1) {
      // ERROR: Copy failed
      return false;
    }
    
    data.adx_value = adx[0];
    return true;
  }
  
  // Calculate Volume MA
  bool CalculateVolume(PriceData& data) {
    long volumes[20];
    if (CopyTickVolume(m_symbol, m_timeframe, 0, 20, volumes) != 20) {
      // ERROR: Not enough history
      return false;
    }
    
    long sum = 0;
    for (int i = 0; i < 20; i++) {
      sum += volumes[i];
    }
    data.volume_ma_20 = (double)sum / 20;
    
    return true;
  }
};

#endif
```

Key Design Decisions:
  ✓ Caching: Prevents recalculating on same bar
  ✓ ADX handle: Created once in constructor
  ✓ Error handling: Returns error flag, doesn't crash
  ✓ Constants: BB_PERIOD, ADX_PERIOD, etc (from spec)
  ✓ Method separation: FetchOHLCV, CalculateBB, etc (testable)

Testing This Module:
  Test_DataProvider_GetCurrentData_ValidData():
    • Call with real EURUSD M30 data
    • Verify BB upper > SMA > BB lower
    • Verify ADX >= 0
    • Verify volume MA > 0
  
  Test_DataProvider_GetCurrentData_InsufficientHistory():
    • Call with symbol/TF that has < 20 bars
    • Verify error flag set
    • Verify function doesn't crash
  
  Test_DataProvider_GetCurrentData_Caching():
    • Call twice same candle
    • Verify cache hit (same object returned)
    • Call next candle
    • Verify cache invalidated (new calculation)

═══════════════════════════════════════════════════════════════════════════════

EXAMPLE 2: PatternRecognizer Module (Medium Complexity)

[PatternRecognizer.mqh - SIMPLIFIED]

```cpp
// PatternRecognizer.mqh
// Purpose: Detect 3-candle band-touch patterns
// Maintained state per symbol per timeframe

class PatternRecognizer {
private:
  string m_symbol;
  int m_timeframe;
  
  // Pattern state (persists across candles)
  int m_pattern_state;  // NO_SIGNAL, SELL_1ST, BUY_1ST, READY_FOR_SELL, READY_FOR_BUY
  datetime m_candle_1_time;
  double m_candle_1_bb_level;
  int m_candle_counter;  // Track which candle in pattern (1, 2, 3)
  
  // Constants from specification
  const double VOLUME_MULTIPLIER_THRESHOLD = 1.5;
  const double ADX_CONSOLIDATION_THRESHOLD = 25.0;
  
public:
  PatternRecognizer(string symbol, int timeframe) {
    m_symbol = symbol;
    m_timeframe = timeframe;
    m_pattern_state = NO_SIGNAL;
    m_candle_counter = 0;
  }
  
  // Check patterns on current bar
  void CheckPatterns(const PriceData& data) {
    // Ignore if data has errors
    if (data.error) {
      m_pattern_state = NO_SIGNAL;
      return;
    }
    
    // State machine for pattern detection
    switch (m_pattern_state) {
      case NO_SIGNAL:
        CheckFirstCandle(data);
        break;
      
      case SELL_1ST_DETECTED:
        ValidateSellSecondCandle(data);
        break;
      
      case BUY_1ST_DETECTED:
        ValidateBuySecondCandle(data);
        break;
      
      case READY_FOR_SELL_ENTRY:
      case READY_FOR_BUY_ENTRY:
        // Pattern ready, waiting for next candle
        // (entry happens in StateMachine, not here)
        break;
    }
  }
  
  int GetPatternState() {
    return m_pattern_state;
  }
  
private:
  void CheckFirstCandle(const PriceData& data) {
    // Check SELL pattern: Price touches upper band
    if (data.high >= data.bb_upper || data.close >= data.bb_upper) {
      m_pattern_state = SELL_1ST_DETECTED;
      m_candle_1_time = iTime(m_symbol, m_timeframe, 0);
      m_candle_1_bb_level = data.bb_upper;
      m_candle_counter = 1;
      return;
    }
    
    // Check BUY pattern: Price touches lower band
    if (data.low <= data.bb_lower || data.close <= data.bb_lower) {
      m_pattern_state = BUY_1ST_DETECTED;
      m_candle_1_time = iTime(m_symbol, m_timeframe, 0);
      m_candle_1_bb_level = data.bb_lower;
      m_candle_counter = 1;
      return;
    }
    
    // No pattern detected
    m_pattern_state = NO_SIGNAL;
  }
  
  void ValidateSellSecondCandle(const PriceData& data) {
    m_candle_counter++;
    
    // Check all conditions for SELL 2nd candle
    
    // 1. Retreat: High must NOT touch upper band again
    if (data.high >= data.bb_upper) {
      m_pattern_state = NO_SIGNAL;  // Failed
      m_candle_counter = 0;
      return;
    }
    
    // 2. Volume: Multiplier < 1.5
    double volume_multiplier = data.volume / data.volume_ma_20;
    if (volume_multiplier >= VOLUME_MULTIPLIER_THRESHOLD) {
      m_pattern_state = NO_SIGNAL;  // High volume, reject
      m_candle_counter = 0;
      return;
    }
    
    // 3. ADX: Must be < 25 (consolidation)
    if (data.adx_value >= ADX_CONSOLIDATION_THRESHOLD) {
      m_pattern_state = NO_SIGNAL;  // Trending, reject
      m_candle_counter = 0;
      return;
    }
    
    // All passed: Ready for entry
    m_pattern_state = READY_FOR_SELL_ENTRY;
  }
  
  void ValidateBuySecondCandle(const PriceData& data) {
    m_candle_counter++;
    
    // Check all conditions for BUY 2nd candle
    
    // 1. Retreat: Low must NOT touch lower band again
    if (data.low <= data.bb_lower) {
      m_pattern_state = NO_SIGNAL;
      m_candle_counter = 0;
      return;
    }
    
    // 2. Volume
    double volume_multiplier = data.volume / data.volume_ma_20;
    if (volume_multiplier >= VOLUME_MULTIPLIER_THRESHOLD) {
      m_pattern_state = NO_SIGNAL;
      m_candle_counter = 0;
      return;
    }
    
    // 3. ADX
    if (data.adx_value >= ADX_CONSOLIDATION_THRESHOLD) {
      m_pattern_state = NO_SIGNAL;
      m_candle_counter = 0;
      return;
    }
    
    // All passed
    m_pattern_state = READY_FOR_BUY_ENTRY;
  }
};
```

Key Features:
  ✓ State machine: Tracks pattern across candles
  ✓ First candle: Detects band touch
  ✓ Second candle: Validates retreat + volume + ADX
  ✓ Ready state: Waits for entry signal
  ✓ Error handling: Resets on any failure
  ✓ Constants: From specification

Testing:
  Test_PatternRecognizer_SELL_Pattern_Valid():
    • Create mock data: 1st touches upper band, 2nd retreats with low volume/ADX
    • Call CheckPatterns() twice
    • Verify state = READY_FOR_SELL_ENTRY
  
  Test_PatternRecognizer_BUY_Pattern_Valid():
    • Similar but for BUY pattern
  
  Test_PatternRecognizer_Pattern_HighVolume_Reject():
    • 1st candle touches band, 2nd has high volume
    • Verify state = NO_SIGNAL (pattern abandoned)

═══════════════════════════════════════════════════════════════════════════════

EXAMPLE 3: StateMachine Module (Orchestration Layer)

[StateMachine.mqh - CORE LOGIC]

The StateMachine is the central orchestrator for ONE symbol. It:
  • Receives price data (primary TF + optional confirm TF)
  • Coordinates all modules
  • Manages state transitions
  • Makes entry/exit decisions
  • Handles news filtering

```cpp
class StateMachine {
private:
  string m_symbol;
  int m_primary_tf;
  int m_confirm_tf;
  
  // Module instances
  DataProvider* m_data_provider;
  DataProvider* m_data_provider_confirm;  // If confirm TF different
  PatternRecognizer* m_pattern_recognizer;
  EntryManager* m_entry_manager;
  ExitCalculator* m_exit_calculator;
  TradeManager* m_trade_manager;
  MultiTimeframeAnalyzer* m_mtf_analyzer;
  Logger* m_logger;
  
  // State
  int m_current_state;  // NO_SIGNAL, SELL_1ST, BUY_1ST, READY_FOR_SELL, READY_FOR_BUY, TRADE_ACTIVE_SELL, TRADE_ACTIVE_BUY
  int m_open_ticket;
  double m_entry_price;
  double m_tp_level;
  double m_sl_level;
  int m_trade_direction;  // SELL or BUY
  
  // Constants
  const int PATTERN_TIMEOUT_BARS = 5;  // Pattern expires after 5 bars of waiting
  
public:
  StateMachine(string symbol, int primary_tf, int confirm_tf,
               DataProvider* data_prov,
               PatternRecognizer* pattern_rec,
               EntryManager* entry_mgr,
               ExitCalculator* exit_calc,
               TradeManager* trade_mgr,
               MultiTimeframeAnalyzer* mtf_analyzer,
               Logger* logger) {
    m_symbol = symbol;
    m_primary_tf = primary_tf;
    m_confirm_tf = confirm_tf;
    
    m_data_provider = data_prov;
    m_pattern_recognizer = pattern_rec;
    m_entry_manager = entry_mgr;
    m_exit_calculator = exit_calc;
    m_trade_manager = trade_mgr;
    m_mtf_analyzer = mtf_analyzer;
    m_logger = logger;
    
    m_current_state = NO_SIGNAL;
    m_open_ticket = -1;
  }
  
  // Main entry point: Called once per candle for this symbol
  void OnCandle(const NewsFilterResult& news) {
    // 1. Get price data
    PriceData primary_data = m_data_provider->GetCurrentData();
    if (primary_data.error) {
      m_logger->LogError(m_symbol, "Failed to fetch primary data");
      return;
    }
    
    // 2. Check news filter FIRST (blocking is critical)
    if (!news.allowed) {
      if (m_current_state == NO_SIGNAL) {
        // Just resting, skip patterns
        return;
      }
      // Pattern in progress, but news blocked
      // Check if we're in READY state: Pattern validated, waiting for 3rd candle entry
      // Entry blocked due to news
      if (m_current_state == READY_FOR_SELL_ENTRY || m_current_state == READY_FOR_BUY_ENTRY) {
        m_current_state = NO_SIGNAL;  // Abandon pattern
        m_logger->LogNewsBlock(m_symbol, news.reason);
      }
      return;
    }
    
    // 3. State machine dispatch
    switch (m_current_state) {
      case NO_SIGNAL:
        HandleNoSignal(primary_data);
        break;
      
      case SELL_1ST_DETECTED:
      case BUY_1ST_DETECTED:
        HandlePatternDetected(primary_data);
        break;
      
      case READY_FOR_SELL_ENTRY:
      case READY_FOR_BUY_ENTRY:
        HandleReadyForEntry(primary_data);
        break;
      
      case TRADE_ACTIVE_SELL:
      case TRADE_ACTIVE_BUY:
        HandleTradeActive(primary_data);
        break;
    }
  }
  
private:
  void HandleNoSignal(const PriceData& data) {
    // Check for pattern start
    m_pattern_recognizer->CheckPatterns(data);
    int pattern_state = m_pattern_recognizer->GetPatternState();
    
    if (pattern_state == READY_FOR_SELL_ENTRY || pattern_state == READY_FOR_BUY_ENTRY) {
      // This shouldn't happen (pattern needs 2 candles minimum)
      return;
    }
    
    if (pattern_state == SELL_1ST_DETECTED) {
      m_current_state = SELL_1ST_DETECTED;
      m_logger->LogInfo(m_symbol, "SELL 1st candle detected");
    } else if (pattern_state == BUY_1ST_DETECTED) {
      m_current_state = BUY_1ST_DETECTED;
      m_logger->LogInfo(m_symbol, "BUY 1st candle detected");
    }
  }
  
  void HandlePatternDetected(const PriceData& data) {
    // Pattern already in progress, check 2nd candle
    m_pattern_recognizer->CheckPatterns(data);
    int pattern_state = m_pattern_recognizer->GetPatternState();
    
    if (pattern_state == READY_FOR_SELL_ENTRY) {
      m_current_state = READY_FOR_SELL_ENTRY;
      m_logger->LogInfo(m_symbol, "SELL pattern ready for entry");
    } else if (pattern_state == READY_FOR_BUY_ENTRY) {
      m_current_state = READY_FOR_BUY_ENTRY;
      m_logger->LogInfo(m_symbol, "BUY pattern ready for entry");
    } else if (pattern_state == NO_SIGNAL) {
      m_current_state = NO_SIGNAL;
      m_logger->LogInfo(m_symbol, "Pattern validation failed");
    }
  }
  
  void HandleReadyForEntry(const PriceData& data) {
    // Pattern ready, entering on this 3rd candle open
    
    EntryResult entry;
    if (m_current_state == READY_FOR_SELL_ENTRY) {
      entry = m_entry_manager->EnterSell(data);
    } else {
      entry = m_entry_manager->EnterBuy(data);
    }
    
    if (entry.success) {
      m_open_ticket = entry.ticket_id;
      m_entry_price = entry.entry_price;
      m_tp_level = entry.tp_level;
      m_sl_level = entry.sl_level;
      m_trade_direction = (m_current_state == READY_FOR_SELL_ENTRY) ? SELL : BUY;
      
      m_current_state = (m_trade_direction == SELL) ? TRADE_ACTIVE_SELL : TRADE_ACTIVE_BUY;
      
      m_logger->LogEntry(m_symbol, entry);
    } else {
      m_current_state = NO_SIGNAL;
      m_logger->LogError(m_symbol, "Entry failed: " + entry.reason);
    }
  }
  
  void HandleTradeActive(const PriceData& data) {
    // Check exits
    ExitSignal exit = m_exit_calculator->CheckExits(m_tp_level, m_sl_level,
                                                     m_trade_direction, data);
    
    if (exit.exit_type != NONE) {
      bool closed = m_trade_manager->ClosePosition(m_open_ticket, exit.exit_price,
                                                   (exit.exit_type == TP_HIT) ? "TP" : "SL");
      
      if (closed) {
        ClosedTrade trade = m_trade_manager->GetLastTrade();
        m_logger->LogExit(m_symbol, trade);
        
        m_current_state = NO_SIGNAL;
        m_open_ticket = -1;
      }
    }
  }
  
public:
  int GetCurrentState() {
    return m_current_state;
  }
  
  bool IsTradeActive() {
    return m_current_state == TRADE_ACTIVE_SELL || m_current_state == TRADE_ACTIVE_BUY;
  }
};
```

Key Points:
  ✓ Coordinates all modules
  ✓ Handles state transitions
  ✓ News filter checked at ALL entry points
  ✓ Pattern validation across candles
  ✓ Trade management (active position monitoring)
  ✓ Logging at every step

Testing:
  Test_StateMachine_SELL_Complete_Flow():
    • Simulate 3-candle pattern
    • Verify state transitions: NO_SIGNAL → SELL_1ST → READY → TRADE_ACTIVE
    • Verify entry placed
  
  Test_StateMachine_News_Block():
    • Simulate pattern detected, then news filter blocks
    • Verify pattern abandoned
    • Verify state returns to NO_SIGNAL
  
  Test_StateMachine_TP_Hit():
    • Simulate open SELL trade
    • Feed data where close <= TP
    • Verify position closed with profit

═══════════════════════════════════════════════════════════════════════════════

2.3 CUSTOM UNIT TESTING FRAMEWORK
─────────────────────────────────────────────────────────────────────────────

For YOUR specific modules, create tests that validate YOUR logic:

[TEST STRUCTURE]

```cpp
// Tests.mqh
// Unit test framework for this EA

#define TEST_ENABLED 1  // Set to 0 to disable all tests in production

void RunAllTests() {
  #if TEST_ENABLED
  
  // DataProvider tests
  Test_DataProvider_GetCurrentData_ValidData();
  Test_DataProvider_GetCurrentData_InsufficientHistory();
  Test_DataProvider_BBCalculation();
  
  // PatternRecognizer tests
  Test_PatternRecognizer_SELLPattern_Valid();
  Test_PatternRecognizer_BUYPattern_Valid();
  Test_PatternRecognizer_VolumeMultiplier();
  Test_PatternRecognizer_ADXThreshold();
  
  // More tests...
  
  #endif
}

// Example: Test DataProvider
void Test_DataProvider_GetCurrentData_ValidData() {
  DataProvider dp("EURUSD", PERIOD_M30);
  
  PriceData data = dp.GetCurrentData();
  
  // Assertions
  assert(data.error == false, "Should not have error");
  assert(data.bb_upper > data.bb_middle, "Upper band > middle");
  assert(data.bb_middle > data.bb_lower, "Middle > lower band");
  assert(data.adx_value >= 0, "ADX >= 0");
  assert(data.volume_ma_20 > 0, "Volume MA > 0");
  
  Print("PASS: Test_DataProvider_GetCurrentData_ValidData");
}

// Example: Test PatternRecognizer
void Test_PatternRecognizer_SELLPattern_Valid() {
  PatternRecognizer pr("EURUSD", PERIOD_M30);
  
  // Create mock data: 1st candle touches upper band
  PriceData data1 = {};
  data1.sma_20 = 1.0800;
  data1.bb_upper = 1.0900;
  data1.bb_lower = 1.0700;
  data1.high = 1.0920;  // Touches upper band
  data1.volume = 100;
  data1.volume_ma_20 = 100;
  data1.adx_value = 15;
  data1.error = false;
  
  pr.CheckPatterns(data1);
  assert(pr.GetPatternState() == SELL_1ST_DETECTED, "Should detect SELL 1st");
  
  // 2nd candle: Retreat, low volume, low ADX
  PriceData data2 = {};
  data2.sma_20 = 1.0800;
  data2.bb_upper = 1.0900;
  data2.bb_lower = 1.0700;
  data2.high = 1.0850;  // Retreats from upper band
  data2.volume = 80;  // Low volume
  data2.volume_ma_20 = 100;
  data2.adx_value = 20;  // ADX < 25
  data2.error = false;
  
  pr.CheckPatterns(data2);
  assert(pr.GetPatternState() == READY_FOR_SELL_ENTRY, "Should be ready for entry");
  
  Print("PASS: Test_PatternRecognizer_SELLPattern_Valid");
}

// Assertion macro
#define assert(condition, message) \
  if (!(condition)) { \
    Print("FAIL: " + message); \
  }
```

Testing Philosophy:
  ✓ Create specific tests for YOUR modules
  ✓ Don't use generic test templates
  ✓ Test YOUR actual logic, not abstract concepts
  ✓ Test boundary conditions (ADX = 25 exactly, volume multiplier = 1.5 exactly)
  ✓ Test error cases (missing data, invalid handles)

═══════════════════════════════════════════════════════════════════════════════

2.4 INTEGRATION TESTING YOUR SPECIFIC FLOWS
─────────────────────────────────────────────────────────────────────────────

Integration tests verify modules work TOGETHER:

[INTEGRATION TEST EXAMPLE: Full SELL Trade Flow]

Test_Integration_SELL_Complete_Flow():
  Step 1: Setup
    • Create all module instances
    • Initialize with EURUSD M30
  
  Step 2: Feed 1st candle data (touches upper band)
    • Data: high ≥ bb_upper
    • Call: DataProvider.GetCurrentData()
    • Call: PatternRecognizer.CheckPatterns()
    • Call: StateMachine.OnCandle()
    • Verify: PatternRecognizer state = SELL_1ST_DETECTED
  
  Step 3: Feed 2nd candle data (validates retreat)
    • Data: high < bb_upper, low volume, ADX < 25
    • Call: StateMachine.OnCandle()
    • Verify: PatternRecognizer state = READY_FOR_SELL_ENTRY
  
  Step 4: Feed 3rd candle data (trigger entry)
    • Data: Normal, no news block, ADX < 25
    • Call: StateMachine.OnCandle()
    • Verify: Entry placed (ticket > 0)
    • Verify: StateMachine state = TRADE_ACTIVE_SELL
    • Verify: TP set to bb_lower
    • Verify: SL set to bb_upper
  
  Step 5: Feed candles until TP hit
    • Data: close ≤ bb_lower
    • Call: StateMachine.OnCandle() multiple times
    • Verify: Position closed
    • Verify: Logger recorded profit
    • Verify: StateMachine state = NO_SIGNAL

Repeat similar test for BUY flow, news blocking, SL hit, etc.

═══════════════════════════════════════════════════════════════════════════════

2.5 SYSTEM TESTING - FULL EA BEHAVIOR
─────────────────────────────────────────────────────────────────────────────

System tests run the full EA with realistic data:

[SYSTEM TEST: Multi-Symbol, Multi-Timeframe]

Test_System_MultiSymbol_ThreeSymbols():
  Setup:
    • Add symbols: EURUSD, EURNZD, GBPUSD
    • Primary TF: M30
    • Confirm TF: H1 (optional)
  
  Simulate:
    • 100 candles of realistic data
    • At each candle: Call OnBar() for all symbols
  
  Verify:
    ☐ Each symbol tracks independently
    ☐ Max 1 position per symbol enforced
    ☐ No position conflicts
    ☐ Entries at correct prices
    ☐ Exits at correct TP/SL levels
    ☐ All trades logged
    ☐ Logging file contains all trades

Test_System_MultiTimeframe_Confirmation():
  Setup:
    • Primary: M30
    • Confirmation: H1
    • Confirmation ENABLED
  
  Scenario 1: M30 signal, H1 agrees
    • M30 detects SELL pattern
    • H1 is in downtrend
    • Verify: Entry allowed
  
  Scenario 2: M30 signal, H1 disagrees
    • M30 detects SELL pattern
    • H1 is in strong uptrend
    • Verify: Entry blocked (H1 doesn't confirm)
  
  Scenario 3: Confirmation disabled
    • M30 detects signal
    • H1 ignored
    • Verify: Entry allowed

Test_System_NewsFilter_BlocksEntries():
  Setup:
    • Load calendar with RBNZ decision
    • Set: Entry window +60 min before, -60 min after
  
  Simulate:
    • Pattern detected 10 min before event
    • Verify: Entry blocked
    • Pattern detected 70 min before
    • Verify: Entry allowed
  
  Verify:
    ☐ Existing trades continue (not closed by news)
    ☐ New entries blocked
    ☐ All blocks logged

User Action at Stage 2:
  ☐ APPROVE - All modules implemented and unit-tested
  ☐ DEBUGGING - Some tests failing, investigating
  ☐ ISSUE - Found implementation problem, redesigning

═══════════════════════════════════════════════════════════════════════════════
STAGE 3: INTEGRATION & SYSTEM TESTING (2-4 Weeks)
═══════════════════════════════════════════════════════════════════════════════

3.1 INTEGRATION TEST EXECUTION
─────────────────────────────────────────────────────────────────────────────

Build up module integration progressively:

[INTEGRATION SEQUENCE]

Week 1:
  ☐ DataProvider + PatternRecognizer
    └─ Feed mock data, verify patterns detected
  
  ☐ DataProvider + ExitCalculator
    └─ Feed positions with known TP/SL, verify exits triggered

Week 2:
  ☐ All 3: DataProvider + PatternRecognizer + ExitCalculator
    └─ Simulate complete trade flow
  
  ☐ Add: EntryManager
    └─ Verify orders placed correctly

Week 3:
  ☐ Add: TradeManager
    └─ Verify positions tracked and closed
  
  ☐ Add: StateMachine
    └─ Verify state transitions work

Week 4:
  ☐ Add: NewsFilter
    └─ Verify entries blocked at correct times
  
  ☐ Add: MultiTimeframeAnalyzer
    └─ Verify confirmation logic works

Final:
  ☐ Full integration: All 10 modules together
    └─ SymbolOrchestrator manages StateMachines

3.2 SYSTEM TESTING - COMPLETE EA
─────────────────────────────────────────────────────────────────────────────

Test the complete EA with realistic simulation:

[FUNCTIONALITY TESTS]

Entry Logic Tests:
  ☐ Entry on 3rd candle open (not close)
  ☐ Entry price matches open[3]
  ☐ TP set to opposite band
  ☐ SL set to entry-side band
  ☐ Order direction correct (SELL/BUY)
  ☐ Volume correct (lot size applied)

Exit Logic Tests:
  ☐ SELL exit: close ≤ TP → close at TP
  ☐ SELL exit: close ≥ SL → close at SL
  ☐ BUY exit: close ≥ TP → close at TP
  ☐ BUY exit: close ≤ SL → close at SL
  ☐ TP takes priority if both hit same bar
  ☐ Position closes immediately (not delayed)

Risk Management Tests:
  ☐ SL always active (protects entry side)
  ☐ SL never trails (static after entry)
  ☐ No partial closes (full position exits)
  ☐ Max 1 position per symbol
  ☐ No "naked" positions (always have TP + SL)

Edge Case Tests (YOUR specific concerns):
  ☐ Market open: Data available?
  ☐ Market close: Positions handled?
  ☐ Overnight gap: BB recalculated?
  ☐ News event: Entry blocked?
  ☐ High volatility: Slippage acceptable?
  ☐ Frozen price: Handled gracefully?
  ☐ Low volume: Volume multiplier correct?

Performance Tests:
  ☐ Main loop < 100ms per symbol per bar
  ☐ Indicator calculations < 5ms each
  ☐ Order execution < 1000ms
  ☐ Memory usage < 50MB
  ☐ No memory leaks (run 1 week simulation)

Multi-Symbol Tests:
  ☐ 1 symbol trades independently
  ☐ 2 symbols: No interference
  ☐ 3 symbols: All trade correctly
  ☐ 5 symbols: All managed correctly
  ☐ Position limits enforced (1 per symbol)

Multi-Timeframe Tests:
  ☐ M30 primary works
  ☐ H1 confirmation works
  ☐ M30 + H1 together work
  ☐ Disable confirmation: M30 only works
  ☐ Enable confirmation: Both required

User Action at Stage 3:
  ☐ APPROVE - All tests pass, ready for code review
  ☐ ISSUE - Tests failing, debugging
  ☐ CONCERN - Performance not meeting targets

═══════════════════════════════════════════════════════════════════════════════
STAGE 4: CODE REVIEW & DOCUMENTATION (1-2 Weeks)
═══════════════════════════════════════════════════════════════════════════════

4.1 CODE REVIEW CHECKLIST
─────────────────────────────────────────────────────────────────────────────

[CUSTOM CODE REVIEW FOR YOUR EA]

Functionality:
  ☐ All 10 modules fully implemented
  ☐ No TODO comments remaining
  ☐ All methods have implementations
  ☐ All YOUR specific features present
  ☐ All error cases handled (no crashes)

Correctness:
  ☐ BB calculation matches MT5 (compare with chart)
  ☐ ADX threshold applied correctly (< 25 for consolidation)
  ☐ Volume multiplier: current / MA correct
  ☐ Entry price: at Open[3] (not Close[2])
  ☐ TP at opposite band (Lower for SELL, Upper for BUY)
  ☐ SL at entry-side band (Upper for SELL, Lower for BUY)
  ☐ News filter: events checked correctly
  ☐ Multi-symbol: independent state machines
  ☐ Multi-timeframe: confirmation logic working

Performance:
  ☐ No unnecessary indicator recalculations
  ☐ Caching implemented (cache same-bar data)
  ☐ No infinite loops
  ☐ Order execution time acceptable
  ☐ Memory usage reasonable

Code Quality:
  ☐ Variable names clear (not a, b, c)
  ☐ Comments explain WHY (not just WHAT)
  ☐ Functions focused (single responsibility)
  ☐ No dead code (unused methods deleted)
  ☐ Consistent style throughout
  ☐ Constants defined (not magic numbers)
  ☐ Error codes documented

Robustness:
  ☐ All error cases handled
  ☐ No crashes on invalid input
  ☐ Graceful degradation (eg: skip entry if data unavailable)
  ☐ Input validation on all external data
  ☐ Recovery from errors

Security:
  ☐ No hardcoded credentials
  ☐ No unsafe memory operations
  ☐ Input validation
  ☐ No SQL injection (if using DB)

Documentation:
  ☐ Each module documented (purpose, responsibilities)
  ☐ Each method has header (inputs, outputs, errors)
  ☐ Design decisions documented (WHY this approach)
  ☐ Architecture documented (how modules work together)
  ☐ Testing documented (how to run tests)
  ☐ Deployment documented (how to attach EA)

4.2 CREATE DOCUMENTATION
─────────────────────────────────────────────────────────────────────────────

[DOCUMENTATION DELIVERABLES]

1. Architecture Document
   ├─ Module overview (10 modules)
   ├─ Data structures (9 structs)
   ├─ Data flows (how data moves through system)
   ├─ State machine (state diagram, transitions)
   ├─ Multi-symbol management (orchestrator pattern)
   ├─ Multi-timeframe logic (confirmation)
   └─ Extension points (future features)

2. Implementation Guide
   ├─ How to compile
     └─ MQL5 editor: File → Compile
     └─ Check for errors/warnings
   
   ├─ How to deploy
     └─ Place .mq5 file in MQL5/Experts folder
     └─ Restart MT5
     └─ Drag EA onto chart
   
   ├─ How to configure
     └─ Input parameters (Symbols, Timeframe, lot size, etc)
     └─ When to enable/disable confirmation
     └─ How to set max positions
   
   ├─ How to test
     └─ Compile and run unit tests
     └─ Review test output
   
   ├─ How to debug
     └─ Enable logging
     └─ Review log file
     └─ Check chart comments

3. Module Documentation (per module)
   ├─ Purpose: What it does
   ├─ Responsibilities: Specific tasks
   ├─ Usage: How to instantiate and call
   ├─ Parameters: What inputs it takes
   ├─ Returns: What outputs it produces
   ├─ Error handling: What can go wrong
   ├─ Testing: How to test this module
   ├─ Dependencies: What it depends on
   └─ Example usage: Code snippet

4. Testing Documentation
   ├─ Unit tests: How to run, what they test
   ├─ Integration tests: Modules together
   ├─ System tests: Full EA behavior
   ├─ Performance tests: Speed and memory
   ├─ Test results: Pass/fail summary
   └─ Known issues: Any test failures and why

5. Troubleshooting Guide
   ├─ Common issues: Specific to YOUR EA
     ├─ "No entries being placed" → Check news filter
     ├─ "Entries on wrong candle" → Check time logic
     ├─ "SL/TP levels incorrect" → Check band calculation
   ├─ Debug steps: How to diagnose problems
   ├─ Log file interpretation: What log messages mean
   └─ Contact: Who to reach for issues

4.3 HANDOFF CHECKLIST
─────────────────────────────────────────────────────────────────────────────

Before moving to Phase 5:

[PHASE 4 COMPLETION CHECKLIST]

Code Complete:
  ☐ All 10 modules implemented (no TODOs)
  ☐ All methods functional
  ☐ All error handling complete
  ☐ No known critical bugs
  ☐ Code review passed

Testing Complete:
  ☐ Unit tests: All pass
  ☐ Integration tests: All pass
  ☐ System tests: All pass
  ☐ Performance tests: Acceptable
  ☐ Edge case tests: Handled

Documentation Complete:
  ☐ Architecture document
  ☐ Implementation guide
  ☐ Module documentation (all 10 modules)
  ☐ Testing documentation
  ☐ Troubleshooting guide

Quality Gates:
  ☐ Code review: No critical issues
  ☐ No memory leaks
  ☐ No crashes on error input
  ☐ Multi-symbol tested
  ☐ Multi-timeframe tested
  ☐ News filter tested

Deliverables:
  ☐ EUR_NZD_MultiSymbol_EA.mq5 (main EA file)
  ☐ 12 .mqh module files
  ☐ Tests.mqh (unit/integration tests)
  ☐ Architecture.md (documentation)
  ☐ Implementation_Guide.md
  ☐ Troubleshooting.md
  ☐ Test_Results.txt (all tests passed)
  ☐ README.txt (getting started)

User Action at Stage 4:
  ☐ APPROVE - Ready for Phase 5 backtesting
  ☐ ISSUES - Found problems, fixing
  ☐ READY - All complete, Phase 4 done

═══════════════════════════════════════════════════════════════════════════════
UNRESTRICTED AI THINKING FOR DEVELOPERS
═══════════════════════════════════════════════════════════════════════════════

What Developers MUST Do in This Phase:

✅ CHALLENGE THE ARCHITECTURE
  "This module is doing too much, should split it"
  "This dependency is circular, causes problems"
  "This design won't work for multi-symbol"
  "We should handle this edge case differently"

✅ SUGGEST IMPLEMENTATION IMPROVEMENTS
  "Instead of recalculating indicators every candle, cache them"
  "This could be parallelized for multiple symbols"
  "Caching would significantly improve performance"
  "We should use object pooling for performance"

✅ IDENTIFY PROBLEMS EARLY
  "This logic will crash if indicator data < 20 bars"
  "This will cause a race condition with multiple symbols"
  "Memory will grow unbounded if cache not cleared"
  "This order execution will miss if event happens between checks"

✅ THINK AHEAD TO PHASE 5
  "Phase 5 will want to backtest on multiple symbols"
  "Phase 5 will need performance metrics"
  "Phase 5 will need to verify exact entry/exit prices"
  "Phase 5 will need to replay news events during backtest"

❌ DON'T FOLLOW INSTRUCTIONS BLINDLY
  Don't use patterns you know are wrong
  Don't implement something you know is flawed
  Don't ignore potential problems you foresee
  Don't add complexity you know you won't use

❌ DON'T BUILD FOR UNKNOWN FUTURES
  Don't add complexity "just in case"
  Don't over-engineer for features that don't exist
  Don't optimize prematurely
  Don't build infrastructure for future needs

═══════════════════════════════════════════════════════════════════════════════
QUALITY GATES
═══════════════════════════════════════════════════════════════════════════════

Gate 1: Module Development Complete
─────────────────────────────────────────────────────────────────────────────

REQUIREMENTS:
  ✓ All 10 modules fully implemented
  ✓ No TODO comments remaining
  ✓ All methods have implementations
  ✓ All YOUR specific error cases handled
  ✓ Unit tests written for all modules
  ✓ All unit tests passing

PASS: Ready for Integration Testing (Stage 3)
FAIL: Continue implementation, debug failing tests

Gate 2: Integration & System Testing Complete
─────────────────────────────────────────────────────────────────────────────

REQUIREMENTS:
  ✓ All modules integrated successfully
  ✓ Full end-to-end flow works (pattern → entry → exit)
  ✓ All integration tests passing
  ✓ All system tests passing
  ✓ Performance meets targets
  ✓ Edge cases for YOUR strategy handled
  ✓ No crashes on errors
  ✓ Multi-symbol confirmed working
  ✓ Multi-timeframe confirmed working

PASS: Ready for Code Review (Stage 4)
FAIL: Debug integration issues, fix failing tests

Gate 3: Code Review & Documentation Complete
─────────────────────────────────────────────────────────────────────────────

REQUIREMENTS:
  ✓ Code review passed (no critical issues)
  ✓ All documentation written
  ✓ Code quality acceptable
  ✓ No technical debt blocking Phase 5
  ✓ Architecture documented
  ✓ Modules documented
  ✓ Tests documented
  ✓ Deployment guide written
  ✓ Troubleshooting guide written

PASS: Ready for Phase 5 Backtesting
FAIL: Address review issues, update documentation

═══════════════════════════════════════════════════════════════════════════════
NEXT STEPS: PHASE 5 BACKTESTING
═══════════════════════════════════════════════════════════════════════════════

After Phase 4 Completion:

Handoff to Phase 5 Team:
  ✓ Source code (all .mq5 and .mqh files)
  ✓ Documentation (architecture, implementation, testing)
  ✓ Compiled EA (.ex5 file)
  ✓ Test results (proof of unit/integration test passage)
  ✓ Known limitations (any edge cases not yet addressed)
  ✓ Performance baselines (expected speed, memory)

Phase 5 Will:
  ✓ Backtest EA on 2+ years of historical data
  ✓ Validate entry/exit prices match specification
  ✓ Validate multi-symbol behavior
  ✓ Validate multi-timeframe logic
  ✓ Validate news filter blocking
  ✓ Calculate performance metrics (Sharpe, Win Rate, Max DD)
  ✓ Forward test on live data (paper trading)
  ✓ Fix any issues found during testing

═══════════════════════════════════════════════════════════════════════════════
END OF PHASE 4 GUIDE
═══════════════════════════════════════════════════════════════════════════════

This guide provides the framework for Phase 4 implementation.
Adapt it to YOUR specific architecture and strategy needs.
The goal is production-ready code ready for backtesting.

Success criteria:
  ✓ All 10 modules implemented
  ✓ All tests passing
  ✓ Code reviewed
  ✓ Documented
  ✓ Ready for Code implementation

═══════════════════════════════════════════════════════════════════════════════
