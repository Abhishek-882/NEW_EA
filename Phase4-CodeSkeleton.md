═══════════════════════════════════════════════════════════════════════════════
PHASE 4: CODE SKELETON & MQL5 TEMPLATE
Multi-Symbol, Multi-Timeframe Mean Reversion EA (v2.0)
═══════════════════════════════════════════════════════════════════════════════

This file provides the code skeleton for developers to fill in with actual MQL5 code.
Follow this structure to implement Phase 4.

═══════════════════════════════════════════════════════════════════════════════
FILE 1: DataStructures.mqh - All Struct Definitions
═══════════════════════════════════════════════════════════════════════════════

```cpp
// DataStructures.mqh
// All data structures for EUR/NZD Multi-Symbol Mean Reversion EA
// Version: 2.0

#ifndef DATASTRUCTURES_H
#define DATASTRUCTURES_H

// ============================================================================
// PRICE DATA STRUCTURE
// ============================================================================

struct PriceData {
  double open;
  double high;
  double low;
  double close;
  long volume;
  double bb_upper;
  double bb_middle;
  double bb_lower;
  double adx_value;
  double volume_ma_20;
  double sma_20;
  bool error;  // True if any calculation failed
};

// ============================================================================
// PATTERN STATUS STRUCTURE
// ============================================================================

enum PatternState {
  NO_SIGNAL = 0,
  SELL_1ST_DETECTED = 1,
  BUY_1ST_DETECTED = 2,
  READY_FOR_SELL_ENTRY = 3,
  READY_FOR_BUY_ENTRY = 4,
  PATTERN_FAILED = 5
};

enum PatternDirection {
  PATTERN_NONE = 0,
  PATTERN_SELL = 1,
  PATTERN_BUY = 2
};

struct PatternStatus {
  PatternState state;
  PatternDirection direction;
  datetime candle_1_time;
  double candle_1_bb_level;
  string validation_reason;
};

// ============================================================================
// NEWS FILTER RESULT
// ============================================================================

struct NewsFilterResult {
  bool allowed;
  string reason;  // "ALLOWED" or "BLOCKED: Event Name in X minutes"
};

// ============================================================================
// ENTRY RESULT
// ============================================================================

struct EntryResult {
  bool success;
  int ticket_id;
  double entry_price;
  datetime entry_time;
  double tp_level;
  double sl_level;
  string symbol;
  string reason;  // Error message if success = false
};

// ============================================================================
// EXIT SIGNAL
// ============================================================================

enum ExitType {
  NONE = 0,
  TP_HIT = 1,
  SL_HIT = 2
};

struct ExitSignal {
  ExitType exit_type;
  double exit_price;
  int exit_bar;
  double pnl_pips;
};

// ============================================================================
// CLOSED TRADE
// ============================================================================

enum TradeDirection {
  TRADE_SELL = 0,
  TRADE_BUY = 1
};

struct ClosedTrade {
  TradeDirection direction;
  datetime entry_time;
  double entry_price;
  datetime exit_time;
  double exit_price;
  double pnl_pips;
  ExitType exit_type;
  int trade_duration_minutes;
  string symbol;
};

// ============================================================================
// CONFIRMATION RESULT
// ============================================================================

enum ConfirmationResult_Type {
  ALLOW = 0,
  BLOCK = 1,
  NEUTRAL = 2
};

struct ConfirmationResult {
  ConfirmationResult_Type result;
  string reason;
};

// ============================================================================
// SYMBOL CONFIGURATION
// ============================================================================

struct SymbolConfig {
  string symbol;
  int primary_timeframe;
  int confirm_timeframe;
  double lot_size;
  bool multi_tf_enabled;
};

#endif  // DATASTRUCTURES_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 2: DataProvider.mqh - Price Data & Indicator Calculations
═══════════════════════════════════════════════════════════════════════════════

```cpp
// DataProvider.mqh
// Fetches OHLCV and calculates indicators (BB, ADX, Volume MA, SMA)
// Per symbol, per timeframe

#ifndef DATAPROVIDER_H
#define DATAPROVIDER_H

#include "DataStructures.mqh"

class DataProvider {
private:
  string m_symbol;
  int m_timeframe;
  int m_handle_adx;
  
  // Caching
  PriceData m_cached_data;
  datetime m_cached_time;
  bool m_cache_valid;
  
  // Constants (from Explained v2.0)
  const int BB_PERIOD = 20;
  const double BB_DEVIATION = 2.0;
  const int ADX_PERIOD = 14;
  const int VOLUME_MA_PERIOD = 20;
  
public:
  DataProvider(string symbol, int timeframe) {
    m_symbol = symbol;
    m_timeframe = timeframe;
    m_cache_valid = false;
    
    // TODO: Create ADX indicator handle
    // m_handle_adx = iADX(...)
  }
  
  ~DataProvider() {
    // TODO: Release ADX handle
  }
  
  PriceData GetCurrentData() {
    // TODO: Check cache (return if valid)
    // TODO: Fetch OHLCV
    // TODO: Calculate Bollinger Bands
    // TODO: Calculate ADX
    // TODO: Calculate Volume MA
    // TODO: Cache result
    // TODO: Return data
    
    PriceData data = {};
    return data;
  }
  
private:
  PriceData FetchOHLCV() {
    PriceData data = {};
    // TODO: Copy Open, High, Low, Close, Volume
    return data;
  }
  
  bool CalculateBollingerBands(PriceData& data) {
    // TODO: Get 20 closes
    // TODO: Calculate SMA-20
    // TODO: Calculate StdDev
    // TODO: Calculate Upper and Lower bands
    return true;
  }
  
  bool CalculateADX(PriceData& data) {
    // TODO: Copy ADX from indicator handle
    return true;
  }
  
  bool CalculateVolume(PriceData& data) {
    // TODO: Get 20 volumes
    // TODO: Calculate average
    return true;
  }
};

#endif  // DATAPROVIDER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 3: PatternRecognizer.mqh - 3-Candle Pattern Detection
═══════════════════════════════════════════════════════════════════════════════

```cpp
// PatternRecognizer.mqh
// Detects 3-candle band-touch patterns (SELL and BUY)

#ifndef PATTERNRECOGNIZER_H
#define PATTERNRECOGNIZER_H

#include "DataStructures.mqh"

class PatternRecognizer {
private:
  string m_symbol;
  int m_timeframe;
  
  // Pattern state (persists across candles)
  PatternState m_pattern_state;
  datetime m_candle_1_time;
  double m_candle_1_bb_level;
  int m_candle_counter;
  
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
  
  void CheckPatterns(const PriceData& data) {
    // TODO: State machine to detect 3-candle patterns
    // Check for SELL (upper band touch)
    // Check for BUY (lower band touch)
  }
  
  PatternState GetPatternState() {
    return m_pattern_state;
  }
  
private:
  void CheckFirstCandle(const PriceData& data) {
    // TODO: Detect band touch (SELL or BUY)
  }
  
  void ValidateSellSecondCandle(const PriceData& data) {
    // TODO: Check retreat, volume, ADX
  }
  
  void ValidateBuySecondCandle(const PriceData& data) {
    // TODO: Check retreat, volume, ADX
  }
};

#endif  // PATTERNRECOGNIZER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 4: NewsFilter.mqh - News Calendar Blocking
═══════════════════════════════════════════════════════════════════════════════

```cpp
// NewsFilter.mqh
// Blocks entries based on news calendar events

#ifndef NEWSFILTER_H
#define NEWSFILTER_H

#include "DataStructures.mqh"

class NewsFilter {
private:
  string m_symbol;
  
  // Calendar data (loaded from file or embedded)
  // TODO: Define structure for calendar events
  
  // Configuration
  const int BLOCKING_WINDOW_BEFORE = 60;  // minutes
  const int BLOCKING_WINDOW_AFTER = 60;   // minutes
  
public:
  NewsFilter(string symbol) {
    m_symbol = symbol;
    // TODO: Load calendar data
  }
  
  bool LoadCalendar(string filename) {
    // TODO: Load news calendar from file
    return true;
  }
  
  NewsFilterResult Check() {
    // TODO: Check if current time is in blocked window
    // TODO: Return ALLOWED or BLOCKED with reason
    
    NewsFilterResult result;
    result.allowed = true;
    result.reason = "ALLOWED";
    return result;
  }
};

#endif  // NEWSFILTER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 5: EntryManager.mqh - Order Entry Placement
═══════════════════════════════════════════════════════════════════════════════

```cpp
// EntryManager.mqh
// Places SELL and BUY orders with static SL/TP

#ifndef ENTRYMANAGER_H
#define ENTRYMANAGER_H

#include "DataStructures.mqh"

class EntryManager {
private:
  string m_symbol;
  double m_lot_size;
  
public:
  EntryManager(string symbol, double lot_size) {
    m_symbol = symbol;
    m_lot_size = lot_size;
  }
  
  EntryResult EnterSell(const PriceData& data) {
    // TODO: Place SELL order
    // TP = Lower Band
    // SL = Upper Band
    // Volume = m_lot_size
    
    EntryResult result;
    result.success = false;
    return result;
  }
  
  EntryResult EnterBuy(const PriceData& data) {
    // TODO: Place BUY order
    // TP = Upper Band
    // SL = Lower Band
    // Volume = m_lot_size
    
    EntryResult result;
    result.success = false;
    return result;
  }
};

#endif  // ENTRYMANAGER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 6: ExitCalculator.mqh - TP/SL Monitoring
═══════════════════════════════════════════════════════════════════════════════

```cpp
// ExitCalculator.mqh
// Monitors static TP and SL levels

#ifndef EXITCALCULATOR_H
#define EXITCALCULATOR_H

#include "DataStructures.mqh"

class ExitCalculator {
private:
  double m_tp_level;
  double m_sl_level;
  TradeDirection m_direction;
  
public:
  ExitCalculator(double tp, double sl, TradeDirection dir) {
    m_tp_level = tp;
    m_sl_level = sl;
    m_direction = dir;
  }
  
  ExitSignal CheckExits(const PriceData& data) {
    // TODO: Check TP hit (opposite band)
    // TODO: Check SL hit (entry-side band)
    // TODO: Return exit signal or NONE
    
    ExitSignal signal;
    signal.exit_type = NONE;
    return signal;
  }
};

#endif  // EXITCALCULATOR_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 7: TradeManager.mqh - Position Lifecycle
═══════════════════════════════════════════════════════════════════════════════

```cpp
// TradeManager.mqh
// Manages open positions and closed trades

#ifndef TRADEMANAGER_H
#define TRADEMANAGER_H

#include "DataStructures.mqh"

class TradeManager {
private:
  string m_symbol;
  
  // Track open positions and closed trades
  // TODO: Define structure for tracking
  
public:
  TradeManager(string symbol) {
    m_symbol = symbol;
  }
  
  bool ClosePosition(int ticket, double exit_price, string exit_reason) {
    // TODO: Close order with ticket_id
    // TODO: Record exit price and reason
    // TODO: Calculate PnL
    return true;
  }
  
  ClosedTrade GetLastTrade() {
    // TODO: Return details of last closed trade
    ClosedTrade trade;
    return trade;
  }
};

#endif  // TRADEMANAGER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 8: MultiTimeframeAnalyzer.mqh - Confirmation Timeframe
═══════════════════════════════════════════════════════════════════════════════

```cpp
// MultiTimeframeAnalyzer.mqh
// Optional confirmation from higher timeframe

#ifndef MULTITIMEFRAMEANALYZER_H
#define MULTITIMEFRAMEANALYZER_H

#include "DataStructures.mqh"

class MultiTimeframeAnalyzer {
private:
  int m_primary_tf;
  int m_confirm_tf;
  
public:
  MultiTimeframeAnalyzer(int primary_tf, int confirm_tf) {
    m_primary_tf = primary_tf;
    m_confirm_tf = confirm_tf;
  }
  
  ConfirmationResult ConfirmSell(const PriceData& confirm_data) {
    // TODO: Check if higher TF allows SELL
    // Don't sell into uptrend
    
    ConfirmationResult result;
    result.result = ALLOW;
    return result;
  }
  
  ConfirmationResult ConfirmBuy(const PriceData& confirm_data) {
    // TODO: Check if higher TF allows BUY
    // Don't buy into downtrend
    
    ConfirmationResult result;
    result.result = ALLOW;
    return result;
  }
};

#endif  // MULTITIMEFRAMEANALYZER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 9: StateMachine.mqh - Per-Symbol Orchestration
═══════════════════════════════════════════════════════════════════════════════

```cpp
// StateMachine.mqh
// Central orchestrator for ONE symbol

#ifndef STATEMACHINE_H
#define STATEMACHINE_H

#include "DataStructures.mqh"

enum StateType {
  STATE_NO_SIGNAL = 0,
  STATE_SELL_1ST = 1,
  STATE_BUY_1ST = 2,
  STATE_READY_FOR_SELL = 3,
  STATE_READY_FOR_BUY = 4,
  STATE_TRADE_ACTIVE_SELL = 5,
  STATE_TRADE_ACTIVE_BUY = 6
};

class StateMachine {
private:
  string m_symbol;
  int m_primary_tf;
  int m_confirm_tf;
  
  // Module instances (pointers, created by SymbolOrchestrator)
  DataProvider* m_data_provider;
  PatternRecognizer* m_pattern_recognizer;
  EntryManager* m_entry_manager;
  ExitCalculator* m_exit_calculator;
  TradeManager* m_trade_manager;
  MultiTimeframeAnalyzer* m_mtf_analyzer;
  
  // State
  StateType m_current_state;
  int m_open_ticket;
  double m_entry_price;
  double m_tp_level;
  double m_sl_level;
  int m_trade_direction;
  
public:
  StateMachine(string symbol, int primary_tf, int confirm_tf) {
    m_symbol = symbol;
    m_primary_tf = primary_tf;
    m_confirm_tf = confirm_tf;
    m_current_state = STATE_NO_SIGNAL;
    m_open_ticket = -1;
  }
  
  void SetModules(DataProvider* dp, PatternRecognizer* pr,
                  EntryManager* em, ExitCalculator* ec,
                  TradeManager* tm, MultiTimeframeAnalyzer* mta) {
    // TODO: Store module pointers
  }
  
  void OnCandle(const NewsFilterResult& news) {
    // TODO: Main state machine logic
    // 1. Get price data
    // 2. Check news filter
    // 3. Dispatch based on current state
    // 4. Transition states
    // 5. Log changes
  }
  
  StateType GetCurrentState() {
    return m_current_state;
  }
  
  bool IsTradeActive() {
    return m_current_state == STATE_TRADE_ACTIVE_SELL ||
           m_current_state == STATE_TRADE_ACTIVE_BUY;
  }
  
private:
  void HandleNoSignal(const PriceData& data) {
    // TODO: Check for pattern start
  }
  
  void HandlePatternDetected(const PriceData& data) {
    // TODO: Validate 2nd candle
  }
  
  void HandleReadyForEntry(const PriceData& data) {
    // TODO: Enter on 3rd candle open
  }
  
  void HandleTradeActive(const PriceData& data) {
    // TODO: Check exits
  }
};

#endif  // STATEMACHINE_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 10: SymbolOrchestrator.mqh - Multi-Symbol Management
═══════════════════════════════════════════════════════════════════════════════

```cpp
// SymbolOrchestrator.mqh
// Manages array of StateMachines (one per symbol)

#ifndef SYMBLORCHESTRATOR_H
#define SYMBLORCHESTRATOR_H

#include "DataStructures.mqh"

class SymbolOrchestrator {
private:
  string m_symbols[];
  StateMachine* m_state_machines[];
  DataProvider* m_data_providers[];
  NewsFilter* m_news_filters[];
  
  int m_symbol_count;
  
  // Configuration
  int m_max_concurrent_positions;
  
public:
  SymbolOrchestrator() {
    m_symbol_count = 0;
    m_max_concurrent_positions = 3;  // 1 per symbol max
  }
  
  void AddSymbol(string symbol, int primary_tf, int confirm_tf, double lot_size) {
    // TODO: Resize arrays
    // TODO: Create module instances for this symbol
    // TODO: Add to tracking arrays
  }
  
  void OnBar(string symbol) {
    // TODO: Find symbol's index
    // TODO: Get price data
    // TODO: Get news filter result
    // TODO: Call state machine
  }
  
  void ProcessAllSymbols() {
    // TODO: Iterate all symbols
    // TODO: Call OnBar for each
  }
  
  int GetOpenPositionCount() {
    // TODO: Count open positions across all symbols
    return 0;
  }
};

#endif  // SYMBLORCHESTRATOR_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 11: Logger.mqh - Trade Logging
═══════════════════════════════════════════════════════════════════════════════

```cpp
// Logger.mqh
// Logs trades, entries, exits, and events

#ifndef LOGGER_H
#define LOGGER_H

#include "DataStructures.mqh"

class Logger {
private:
  string m_log_filename;
  
public:
  Logger() {
    // TODO: Initialize log file path
  }
  
  void LogEntry(string symbol, const EntryResult& entry) {
    // TODO: Log format: "[SYMBOL] [TIME] SELL/BUY entered at X, TP Y, SL Z"
    // TODO: Write to file and chart comment
  }
  
  void LogExit(string symbol, const ClosedTrade& trade) {
    // TODO: Log format: "[SYMBOL] [TIME] SELL/BUY closed: TP/SL hit, PnL"
    // TODO: Write to file and chart comment
  }
  
  void LogNewsBlock(string symbol, string event) {
    // TODO: Log format: "[SYMBOL] Entry blocked by EVENT"
    // TODO: Write to file
  }
  
  void LogInfo(string symbol, string message) {
    // TODO: General logging
  }
  
  void LogError(string symbol, string message) {
    // TODO: Error logging
  }
};

#endif  // LOGGER_H
```

═══════════════════════════════════════════════════════════════════════════════
FILE 12: Main EA File - EUR_NZD_MultiSymbol_EA.mq5
═══════════════════════════════════════════════════════════════════════════════

```cpp
// EUR_NZD_MultiSymbol_EA.mq5
// Multi-Symbol, Multi-Timeframe Mean Reversion Trading EA
// Version: 2.0
// Strategy: Bollinger Bands + ADX + Volume + News Filter

#property copyright "Your Name"
#property version "2.0"
#property description "Mean Reversion EA: Bollinger Bands + ADX + News Filter"
#property strict

// Include all modules
#include "DataStructures.mqh"
#include "DataProvider.mqh"
#include "PatternRecognizer.mqh"
#include "NewsFilter.mqh"
#include "EntryManager.mqh"
#include "ExitCalculator.mqh"
#include "TradeManager.mqh"
#include "StateMachine.mqh"
#include "SymbolOrchestrator.mqh"
#include "MultiTimeframeAnalyzer.mqh"
#include "Logger.mqh"

// Global instances
SymbolOrchestrator* g_orchestrator;
Logger* g_logger;
NewsFilter* g_news_filter[];

// Input parameters
input string Symbols = "EURUSD,EURNZD,GBPUSD";  // Comma-separated symbols
input int PrimaryTimeframe = PERIOD_M30;
input int ConfirmTimeframe = PERIOD_H1;
input bool UseMultiTimeframeConfirmation = false;
input double LotSize = 0.1;
input bool EnableLogging = true;

// ============================================================================
// INITIALIZATION
// ============================================================================

int OnInit() {
  // TODO: Initialize global objects
  // g_orchestrator = new SymbolOrchestrator();
  // g_logger = new Logger();
  
  // TODO: Parse symbols from input string
  // TODO: Add each symbol to orchestrator
  
  // TODO: Load news calendar
  
  return INIT_SUCCEEDED;
}

// ============================================================================
// MAIN TRADING LOOP
// ============================================================================

void OnBar() {
  // Called on each bar close for the chart symbol
  
  // TODO: Process all symbols
  // g_orchestrator->ProcessAllSymbols();
  
  // TODO: Check position limits
  
  // TODO: Log status if needed
}

void OnTick() {
  // Optional: Real-time monitoring
  // Can be used for intermediate updates
  // Most logic should be in OnBar()
}

// ============================================================================
// CLEANUP
// ============================================================================

void OnDeinit(const int reason) {
  // TODO: Cleanup (optional save state)
  // Delete g_orchestrator;
  // Delete g_logger;
}

// ============================================================================
// TEST/DEBUG FUNCTIONS
// ============================================================================

void OnStart() {
  // For testing in Expert (not attached to chart)
  // Use for unit tests
}
```

═══════════════════════════════════════════════════════════════════════════════
IMPLEMENTATION CHECKLIST
═══════════════════════════════════════════════════════════════════════════════

Priority Order (Layer by Layer):

Layer 0:
  ☐ DataStructures.mqh (all structs)
  ☐ NewsCalendarData.mqh (embedded data)

Layer 1:
  ☐ DataProvider.mqh (implement GetCurrentData, BB, ADX, Volume MA)
  ☐ NewsFilter.mqh (implement Check, LoadCalendar)
  ☐ Logger.mqh (implement logging methods)

Layer 2:
  ☐ PatternRecognizer.mqh (implement CheckPatterns state machine)
  ☐ ExitCalculator.mqh (implement CheckExits)

Layer 3:
  ☐ EntryManager.mqh (implement EnterSell, EnterBuy)
  ☐ MultiTimeframeAnalyzer.mqh (implement Confirm methods)

Layer 4:
  ☐ TradeManager.mqh (implement ClosePosition, GetLastTrade)

Layer 5:
  ☐ StateMachine.mqh (implement OnCandle, HandleStates)

Layer 6:
  ☐ SymbolOrchestrator.mqh (implement AddSymbol, OnBar, ProcessAllSymbols)

Layer 7:
  ☐ Main EA file (OnInit, OnBar, OnDeinit)

Testing:
  ☐ Unit tests for each module
  ☐ Integration tests (modules together)
  ☐ System tests (full EA)
  ☐ Performance tests

Documentation:
  ☐ Architecture document
  ☐ Implementation guide
  ☐ Module documentation
  ☐ Testing documentation

═══════════════════════════════════════════════════════════════════════════════
KEY IMPLEMENTATION NOTES
═══════════════════════════════════════════════════════════════════════════════

1. Caching Strategy
   • DataProvider caches data (check bar time, return if same)
   • Don't recalculate BB, ADX, Volume MA every call
   • Invalidate cache on new bar

2. Error Handling
   • No crashes (always return error code)
   • Log errors but continue
   • Graceful degradation

3. Multi-Symbol Management
   • Each symbol has independent state machine
   • No cross-symbol interference
   • Max 1 position per symbol

4. Multi-Timeframe Logic
   • Optional confirmation (can be disabled)
   • Higher TF must "agree" before entry
   • Use SMA-200 on higher TF as reference

5. News Filter Integration
   • Check at 3 points: 1st candle, 2nd candle, 3rd candle entry
   • Block entries inside event windows
   • Don't close existing trades

6. State Machine
   • 7 states total
   • Transitions on candle close
   • Pattern timeout: Abandon after 5 bars waiting

═══════════════════════════════════════════════════════════════════════════════
END OF CODE SKELETON
═══════════════════════════════════════════════════════════════════════════════
