═══════════════════════════════════════════════════════════════════════════════
UPDATED STRATEGY SPECIFICATION: BB + ADX + Volume + News Filter
MEAN REVERSION WITH UPDATED MECHANICS
═══════════════════════════════════════════════════════════════════════════════

SPECIFICATION VERSION: 2.0 (UPDATED)
Date: January 13, 2026
Status: LOCKED - Ready for Phase 4 Implementation

═══════════════════════════════════════════════════════════════════════════════
EXECUTIVE SUMMARY - STRATEGY OVERVIEW
═══════════════════════════════════════════════════════════════════════════════

STRATEGY TYPE: Mean Reversion with Multi-Symbol, Multi-Timeframe Support

Core Logic:
  ✓ Enters when price spikes into a Bollinger Band (upper/lower)
  ✓ In a low-ADX consolidation environment (ADX < 25)
  ✓ With muted volume (current volume < 1.5× its 20-bar average)
  ✓ And no major news events within a defined window
  ✓ Exits at opposite Bollinger Band (static target)
  ✓ SL at same-side Bollinger Band (protection level)

NEW IN VERSION 2.0:
  ✓ Multi-timeframe support (simultaneously analyze multiple timeframes)
  ✓ Multi-symbol support (trade multiple currency pairs at once)
  ✓ Simplified SL mechanics (BB-based instead of ATR-based)
  ✓ Trailing mechanism only for TP (not for SL)
  ✓ Static SL at entry-side band (never trails)

═══════════════════════════════════════════════════════════════════════════════
1. CORE BUILDING BLOCKS
═══════════════════════════════════════════════════════════════════════════════

1.1 BOLLINGER BANDS - PRIMARY SIGNAL
────────────────────────────────────────────────────────────────────────────────

Configuration:
  Period: N = 20
  Deviation: 2.0
  Applied to: Close prices
  Timeframe: Primary (e.g., M30, H1, D1 - configurable)

Formulas:
  Middle Band (SMA) = SMA₂₀(Close)
  Upper Band = SMA₂₀(Close) + 2 × StdDev₂₀(Close)
  Lower Band = SMA₂₀(Close) - 2 × StdDev₂₀(Close)

Roles:
  Upper Band:
    └─ Resistance zone for SELL setups
    └─ Price touching/closing above = overbought condition
    └─ Entry point: SELL at 3rd candle open
    └─ Exit point: SELL closes at Lower Band = TP hit
    └─ SL level: Upper Band (same side as entry)

  Lower Band:
    └─ Support zone for BUY setups
    └─ Price touching/closing below = oversold condition
    └─ Entry point: BUY at 3rd candle open
    └─ Exit point: BUY closes at Upper Band = TP hit
    └─ SL level: Lower Band (same side as entry)

  Middle Band (SMA-20):
    └─ Equilibrium/average price
    └─ Reference for trailing TP mechanism (optional)

1.2 ADX - CONSOLIDATION DETECTOR (REVERSED ROLE)
────────────────────────────────────────────────────────────────────────────────

Period: 14
Threshold: 25

Interpretation (DELIBERATELY REVERSED):
  ADX < 25:
    ├─ Status: Consolidation / compressed market
    ├─ Meaning: Signal-POSITIVE for this strategy
    └─ Reason: Price coiled, likely to revert from band toward mean

  ADX ≥ 25:
    ├─ Status: Trending market
    ├─ Meaning: Signal-NEGATIVE
    └─ Reason: Move into band more likely trend continuation, not reversion

ADX Checks (3-Candle Pattern):
  ├─ 2nd Candle Close: ADX must be < 25 (consolidation confirmed)
  ├─ 3rd Candle Open: ADX must still be < 25 (no trend break)
  └─ If ADX ≥ 25 at any check: Signal rejected (entry blocked)

1.3 VOLUME - MULTIPLIER METHOD
─────────────────────────────────────────────────────────────────────────────

Lookback: 20 candles
Calculation:
  AvgVol₂₀ = (1/20) × Σ(Volume[i] for i=0..19)
  Multiplier = Current_Volume / AvgVol₂₀

Condition (2nd Candle):
  Multiplier < 1.5 = PASS (volume muted, consolidation)
  Multiplier ≥ 1.5 = FAIL (volume high, potential breakout, skip signal)

Interpretation:
  ├─ Low volume confirms the band spike is not a strong breakout
  ├─ Supports reversal expectation (revert to mean)
  └─ Filter prevents entries during momentum-driven moves

1.4 NEWS FILTER - CALENDAR-BASED BLOCKING
──────────────────────────────────────────────────────────────────────────────

Data Source:
  ├─ Pre-loaded news calendar (EUR/NZD events 2023-2026+)
  ├─ Static data, updated periodically
  ├─ GMT timestamps for all events
  └─ Impact levels: Critical, High, Medium

Blocking Rules:
  Window: ±60 minutes around event time (configurable)
  Impact Filter: Critical + High only
  Symbol Filter: EUR or NZD related events

Logic:
  ├─ At each candle: Check if current time is inside ANY blocked window
  ├─ If yes: Block new entries (SELL and BUY)
  ├─ If no: Entries allowed
  └─ Existing trades: Continue normally (no forced close)

Examples of Blocking Events:
  ├─ RBNZ Rate Decision
  ├─ ECB Rate Decision
  ├─ EUR/NZD GDP announcements
  ├─ CPI releases (EUR or NZD)
  ├─ Employment data (EUR or NZD)
  └─ Any Critical impact event

═══════════════════════════════════════════════════════════════════════════════
2. MULTI-TIMEFRAME & MULTI-SYMBOL SUPPORT
═══════════════════════════════════════════════════════════════════════════════

2.1 MULTI-TIMEFRAME CAPABILITY
────────────────────────────────────────────────────────────────────────────────

Architecture:
  ├─ Primary Timeframe: Where 3-candle pattern is detected (main signal)
  ├─ Confirmation Timeframe: Optional higher timeframe for validation
  └─ Both operate independently but can reinforce each other

Primary Timeframe Options:
  M30, H1, H4, D1 (user configurable)

Confirmation Timeframe (if enabled):
  ├─ Must be HIGHER than primary (e.g., if primary=M30, use H1+)
  ├─ SELL: Check if Higher TF is in downtrend or consolidation
  ├─ BUY: Check if Higher TF is in uptrend or consolidation
  └─ If confirmation enabled: Higher TF must "agree" before entry

Implementation:
  ├─ Each timeframe has its own PatternRecognizer instance
  ├─ Each timeframe calculates its own indicators
  ├─ StateMachine can accept multi-timeframe signal inputs
  └─ Entry only if: Primary pattern + (confirmation allows OR confirmation disabled)

2.2 MULTI-SYMBOL CAPABILITY
────────────────────────────────────────────────────────────────────────────────

Architecture:
  ├─ Support trading multiple symbols simultaneously
  ├─ Each symbol has independent state machine
  ├─ Each symbol tracked separately (different tickets, different state)
  └─ Max concurrent positions: 1 per symbol (configurable)

Supported Symbols:
  EUR/NZD (primary), plus:
  ├─ EUR/USD
  ├─ USD/JPY
  ├─ GBP/USD
  ├─ Any other forex pair (user configurable)

Implementation:
  ├─ Create array of StateMachines (one per symbol)
  ├─ Create separate news filter per symbol (or shared with symbol-aware filtering)
  ├─ Track positions per symbol independently
  ├─ OnBar() processes all symbols' state machines
  └─ Tickets stored in symbol-indexed array

Example:
  If trading {EUR/NZD, EUR/USD, GBP/USD}:
    ├─ EUR/NZD: Open SELL at 1.5200
    ├─ EUR/USD: Waiting for 1st candle
    ├─ GBP/USD: Open BUY at 1.2500
    └─ Each managed independently

═══════════════════════════════════════════════════════════════════════════════
3. SELL SETUP - UPPER BAND MEAN REVERSION
═══════════════════════════════════════════════════════════════════════════════

3.1 CANDLE ROLES (3-CANDLE PATTERN)
────────────────────────────────────────────────────────────────────────────────

Candle 1: Band Touch Candle
  ├─ High ≥ Upper_Band OR Close ≥ Upper_Band
  ├─ Represents spike into overbought territory
  └─ Record: Upper Band level at this close

Candle 2: Validation Candle
  ├─ Must NOT re-touch upper band
  ├─ Must show low volume (multiplier < 1.5)
  ├─ Must confirm low ADX (< 25)
  └─ Represents market backing off from extreme

Candle 3: Entry Candle
  ├─ Enter SELL at Open price
  ├─ Re-validate ADX < 25 at open
  ├─ Re-validate News filter at open
  └─ If all pass: Place order

3.2 STEP-BY-STEP SELL LOGIC
────────────────────────────────────────────────────────────────────────────────

Pre-Step: News Filter Check (ALWAYS first)
  ├─ At every new candle: Check if current time is in blocked window
  ├─ If BLOCKED: Skip all logic (no new SELL signals this candle)
  ├─ If ALLOWED: Proceed to pattern logic

Step 1: Detect 1st Candle (Band Touch)
  Condition:
    High[1] ≥ Upper_Band[1] OR Close[1] ≥ Upper_Band[1]
  
  Action (if true):
    ├─ Mark: 1st_Candle detected
    ├─ Record: candle_1_time, Upper_Band_value
    ├─ State: SIGNAL_DETECTED
    └─ Wait for 2nd candle

Step 2: Validate 2nd Candle
  Condition (ALL must be true):
    a) Band Retreat: High[2] < Upper_Band[2]
    b) Volume: (Volume[2] / AvgVol₂₀) < 1.5
    c) ADX: ADX[2] < 25
  
  Action (if all pass):
    ├─ State: READY_FOR_ENTRY
    └─ Wait for 3rd candle open
  
  Action (if any fails):
    ├─ State: NO_SIGNAL
    └─ Pattern abandoned, wait for new 1st candle

Step 3: Final Check & Entry on 3rd Candle Open
  Final Validations (at 3rd candle OPEN):
    a) ADX < 25? (re-check: market still consolidating)
    b) News filter still ALLOWED? (timing didn't slip into window)
  
  Action (if both pass):
    ├─ Place SELL order at Open[3]
    ├─ Volume: lot_size (e.g., 0.1)
    ├─ TP: Lower_Band (current value at entry)
    ├─ SL: Upper_Band (current value at entry)
    ├─ State: TRADE_ACTIVE
    └─ Record ticket, entry price, TP level, SL level
  
  Action (if any fails):
    ├─ State: NO_SIGNAL
    └─ Entry rejected, wait for next pattern

═══════════════════════════════════════════════════════════════════════════════
4. BUY SETUP - LOWER BAND MEAN REVERSION (MIRROR OF SELL)
═══════════════════════════════════════════════════════════════════════════════

4.1 BUY LOGIC (MIRROR PATTERN)
────────────────────────────────────────────────────────────────────────────────

Pre-Step: News Filter Check
  ├─ Check if current time in blocked window
  ├─ If BLOCKED: Skip (no BUY signals)
  └─ If ALLOWED: Proceed

Step 1: Detect 1st Candle
  Condition:
    Low[1] ≤ Lower_Band[1] OR Close[1] ≤ Lower_Band[1]
  
  Action (if true):
    ├─ Mark: 1st_Candle (BUY side)
    ├─ Record: candle_1_time, Lower_Band_value
    └─ State: SIGNAL_DETECTED

Step 2: Validate 2nd Candle
  Condition (ALL must be true):
    a) Band Retreat: Low[2] > Lower_Band[2]
    b) Volume: (Volume[2] / AvgVol₂₀) < 1.5
    c) ADX: ADX[2] < 25
  
  Action (if all pass):
    ├─ State: READY_FOR_ENTRY
    └─ Wait for 3rd candle
  
  Action (if any fails):
    ├─ State: NO_SIGNAL
    └─ Pattern abandoned

Step 3: Entry on 3rd Candle Open
  Final Validations:
    a) ADX < 25?
    b) News filter ALLOWED?
  
  Action (if both pass):
    ├─ Place BUY order at Open[3]
    ├─ Volume: lot_size
    ├─ TP: Upper_Band (current at entry)
    ├─ SL: Lower_Band (current at entry)
    ├─ State: TRADE_ACTIVE
    └─ Record entry details
  
  Action (if any fails):
    ├─ State: NO_SIGNAL
    └─ Entry rejected

═══════════════════════════════════════════════════════════════════════════════
5. TRADE MANAGEMENT - EXIT MECHANICS
═══════════════════════════════════════════════════════════════════════════════

5.1 TAKE PROFIT (STATIC TARGET)
────────────────────────────────────────────────────────────────────────────────

SELL Trade TP:
  ├─ Target: Lower_Band (opposite side)
  ├─ Entry: SELL at High[3] or Open[3]
  ├─ TP Level: Lower_Band value at entry time (FIXED)
  ├─ Exit Condition: Close ≤ TP_Level
  └─ Exit Action: Close SELL with profit

BUY Trade TP:
  ├─ Target: Upper_Band (opposite side)
  ├─ Entry: BUY at Low[3] or Open[3]
  ├─ TP Level: Upper_Band value at entry time (FIXED)
  ├─ Exit Condition: Close ≥ TP_Level
  └─ Exit Action: Close BUY with profit

TP Trailing (OPTIONAL - for profit protection):
  ├─ NOT a stop loss mechanism
  ├─ Purpose: Lock in partial profits on extended moves
  ├─ Implementation: Once price exceeds TP by X%, trail at SMA-20
  ├─ Example SELL: If Close drops below TP, DON'T exit yet
  │  └─ Instead: Trail TP down using SMA-20 (lock profits, catch more)
  └─ Example BUY: If Close rises above TP, trail TP up using SMA-20

Note: TP trailing is OPTIONAL feature, not required for core logic

5.2 STOP LOSS (STATIC, OPPOSITE SIDE BAND)
──────────────────────────────────────────────────────────────────────────────

CRITICAL CHANGE FROM V1.0:
  ✓ NO ATR-based stop loss
  ✓ SL = Same-side Bollinger Band (entry-side band)
  ✓ SL is STATIC (never changes after entry)
  ✓ SL acts as hard limit (position closes on touch)

SELL Trade SL:
  ├─ SL Level: Upper_Band (same side as entry)
  ├─ Entry: Went above Upper Band, now selling
  ├─ SL: Back above Upper Band = trade stopped
  ├─ Exit Condition: Close ≥ SL_Level
  └─ Exit Action: Close SELL with loss

BUY Trade SL:
  ├─ SL Level: Lower_Band (same side as entry)
  ├─ Entry: Went below Lower Band, now buying
  ├─ SL: Back below Lower Band = trade stopped
  ├─ Exit Condition: Close ≤ SL_Level
  └─ Exit Action: Close BUY with loss

SL Behavior:
  ├─ Set at entry, NEVER modified
  ├─ Checked on every candle (each close)
  ├─ If hit: Immediate close (priority over TP if both hit same bar)
  ├─ NO partial closes (full position exits)
  └─ On hit: Transition to NO_SIGNAL state

5.3 EXIT PRIORITY
──────────────────────────────────────────────────────────────────────────────

If Multiple Exits Possible on Same Bar:
  1. TP HIT: Close with profit (primary priority)
  2. SL HIT: Close with loss (secondary)
  3. Both on same bar: TP prioritized, position closes at TP level

Example SELL Trade:
  ├─ Entered at 1.5250, TP at 1.5200, SL at 1.5300
  ├─ Bar opens: High 1.5310 (touches SL)
  ├─ Bar closes: Low 1.5195 (touches TP)
  └─ Result: Close at TP (1.5200) = profit priority

═══════════════════════════════════════════════════════════════════════════════
6. STATE MACHINE - OPERATIONAL FLOW
═══════════════════════════════════════════════════════════════════════════════

6.1 STATE DEFINITIONS
────────────────────────────────────────────────────────────────────────────────

NO_SIGNAL (Resting State)
  ├─ Waiting for 1st candle band touch
  ├─ News filter checked at every candle
  ├─ Check both SELL (upper) and BUY (lower) patterns
  └─ Transition: On band touch → SELL_1ST or BUY_1ST

SELL_1ST_DETECTED
  ├─ 1st candle SELL pattern detected (touched Upper Band)
  ├─ Waiting for 2nd candle to validate
  ├─ Transition (if 2nd validates): → READY_FOR_SELL_ENTRY
  └─ Transition (if 2nd fails): → NO_SIGNAL

BUY_1ST_DETECTED
  ├─ 1st candle BUY pattern detected (touched Lower Band)
  ├─ Waiting for 2nd candle to validate
  ├─ Transition (if 2nd validates): → READY_FOR_BUY_ENTRY
  └─ Transition (if 2nd fails): → NO_SIGNAL

READY_FOR_SELL_ENTRY
  ├─ 2nd candle validated (retreat, volume, ADX OK)
  ├─ Waiting for 3rd candle open to enter
  ├─ Transition (if 3rd entry OK): → TRADE_ACTIVE (SELL)
  └─ Transition (if 3rd entry blocked): → NO_SIGNAL

READY_FOR_BUY_ENTRY
  ├─ 2nd candle validated
  ├─ Waiting for 3rd candle open to enter
  ├─ Transition (if 3rd entry OK): → TRADE_ACTIVE (BUY)
  └─ Transition (if 3rd entry blocked): → NO_SIGNAL

TRADE_ACTIVE_SELL
  ├─ SELL position open
  ├─ Monitor TP (Lower Band) and SL (Upper Band) on each candle
  ├─ Check for exit conditions every close
  └─ Transition (on exit): → CLOSED → NO_SIGNAL

TRADE_ACTIVE_BUY
  ├─ BUY position open
  ├─ Monitor TP (Upper Band) and SL (Lower Band) on each candle
  ├─ Check for exit conditions every close
  └─ Transition (on exit): → CLOSED → NO_SIGNAL

6.2 STATE TRANSITION DIAGRAM
────────────────────────────────────────────────────────────────────────────────

                         NO_SIGNAL (Resting)
                        /           \
                    [1st touch]   [1st touch]
                    /               \
        SELL_1ST_DETECTED      BUY_1ST_DETECTED
            /    \               /    \
        [valid] [fail]      [valid] [fail]
        /        \           /        \
   READY_SELL  NO_SIGNAL  READY_BUY  NO_SIGNAL
      / \                  / \
   [OK] [block]        [OK] [block]
   /     \              /     \
TRADE_ACTIVE_SELL  NO_SIGNAL  TRADE_ACTIVE_BUY  NO_SIGNAL
   / \                                    / \
[TP/SL]                                [TP/SL]
 /   \                                  /   \
CLOSED → NO_SIGNAL                  CLOSED → NO_SIGNAL

6.3 NEWS FILTER INTEGRATION (3 CRITICAL POINTS)
────────────────────────────────────────────────────────────────────────────────

Point 1: 1st Candle Detection (NO_SIGNAL → 1ST_DETECTED)
  ├─ Check news filter: Is current time blocked?
  ├─ If BLOCKED: Don't even start pattern (skip to next candle)
  ├─ If ALLOWED: Allow band-touch detection
  └─ Effect: Prevents patterns from forming during risky windows

Point 2: 2nd Candle Validation (1ST_DETECTED → READY_FOR_ENTRY)
  ├─ Even if price/volume/ADX criteria pass
  ├─ If 2nd candle close is inside blocked window: Reject
  └─ State: Return to NO_SIGNAL (pattern abandoned)

Point 3: 3rd Candle Entry (READY_FOR_ENTRY → TRADE_ACTIVE)
  ├─ Final check before order placement
  ├─ If NOW in blocked window (time moved closer to event): No entry
  ├─ State: Return to NO_SIGNAL
  └─ Effect: Last-minute protection against news events

═══════════════════════════════════════════════════════════════════════════════
7. HOW MULTI-TIMEFRAME & MULTI-SYMBOL WORK TOGETHER
═══════════════════════════════════════════════════════════════════════════════

7.1 MULTI-TIMEFRAME EXAMPLE
────────────────────────────────────────────────────────────────────────────────

Setup: Primary M30, Confirmation H1

Scenario 1 - Both Align (Best case):
  ├─ M30: 1st candle touches Upper Band (SELL signal)
  ├─ H1: Already in downtrend or consolidation
  └─ Result: HIGH confidence SELL entry

Scenario 2 - Conflict:
  ├─ M30: 1st candle touches Upper Band (SELL signal)
  ├─ H1: In strong uptrend
  └─ Result: BLOCK entry (higher TF says don't sell into uptrend)

Scenario 3 - Confirmation disabled:
  ├─ M30: 1st candle touches (SELL signal)
  ├─ H1: Ignored (confirmation off)
  └─ Result: ALLOW entry (M30 logic sufficient)

7.2 MULTI-SYMBOL EXAMPLE
────────────────────────────────────────────────────────────────────────────────

Scenario: Trading EUR/NZD, EUR/USD, GBP/USD on M30

Timeline:
  ├─ 10:00 UTC: EUR/NZD 1st candle touches Upper Band
  │            → State: SELL_1ST_DETECTED
  ├─ 10:30 UTC: EUR/USD 1st candle touches Lower Band
  │            → State: BUY_1ST_DETECTED
  ├─ 10:30 UTC: EUR/NZD 2nd candle validates
  │            → State: READY_FOR_SELL_ENTRY (waiting for 3rd candle open)
  ├─ 11:00 UTC: EUR/NZD 3rd candle open, enters SELL
  │            → State: TRADE_ACTIVE_SELL
  ├─ 11:00 UTC: EUR/USD 2nd candle validates
  │            → State: READY_FOR_BUY_ENTRY
  ├─ 11:30 UTC: EUR/USD 3rd candle open, enters BUY
  │            → State: TRADE_ACTIVE_BUY
  ├─ 12:00 UTC: EUR/NZD SELL hits TP
  │            → State: CLOSED (profit)
  ├─ 12:00 UTC: EUR/NZD returns to NO_SIGNAL (ready for next pattern)
  ├─ 13:30 UTC: EUR/USD BUY hits SL
  │            → State: CLOSED (loss)
  └─ 13:30 UTC: EUR/USD returns to NO_SIGNAL

Result: Each symbol trades independently, simultaneous management possible

═══════════════════════════════════════════════════════════════════════════════
8. COMPLETE ENTRY/EXIT RULES SUMMARY
═══════════════════════════════════════════════════════════════════════════════

ENTRY CONDITIONS (Must ALL pass):
  ├─ 1st Candle: Price touches band (High ≥ UB for SELL, Low ≤ LB for BUY)
  ├─ 2nd Candle: Retreat, low volume, ADX < 25
  ├─ 3rd Candle: ADX still < 25, news filter ALLOWED
  └─ Result: Entry at Open[3]

EXIT CONDITIONS (Checked every candle):
  ├─ SELL exits:
  │  ├─ TP hit: Close ≤ Lower_Band → Close with profit
  │  └─ SL hit: Close ≥ Upper_Band → Close with loss
  ├─ BUY exits:
  │  ├─ TP hit: Close ≥ Upper_Band → Close with profit
  │  └─ SL hit: Close ≤ Lower_Band → Close with loss
  └─ TP takes priority if both hit same bar

═══════════════════════════════════════════════════════════════════════════════
9. CONFIGURATION PARAMETERS
═══════════════════════════════════════════════════════════════════════════════

User-Configurable Settings:

Bollinger Bands:
  ├─ Period: 20 (fixed)
  ├─ Deviation: 2.0 (fixed)
  └─ Applied to: Close prices

ADX:
  ├─ Period: 14 (fixed)
  └─ Threshold: 25 (fixed)

Volume:
  ├─ MA Period: 20 (fixed)
  └─ Multiplier Threshold: 1.5 (fixed)

News Filter:
  ├─ Pre-event window: 60 minutes (configurable)
  ├─ Post-event window: 60 minutes (configurable)
  ├─ Impact filter: Critical + High (configurable)
  └─ Calendar: EUR/NZD events (embedded, updatable)

Multi-Timeframe:
  ├─ Primary timeframe: M30, H1, H4, D1 (configurable)
  ├─ Confirmation timeframe: Optional (configurable)
  └─ Confirmation mode: ON/OFF (configurable)

Multi-Symbol:
  ├─ Symbols to trade: User list (configurable)
  ├─ Max positions per symbol: 1 (fixed)
  └─ Max total positions: (# symbols) × 1 (configurable)

Position Management:
  ├─ Lot size: Fixed (configurable per symbol)
  └─ No dynamic sizing

═══════════════════════════════════════════════════════════════════════════════
10. WHAT THIS STRATEGY IS AND IS NOT
═══════════════════════════════════════════════════════════════════════════════

IT IS:
  ✓ A Bollinger Band mean-reversion system
  ✓ For multi-symbol, multi-timeframe trading
  ✓ Using 3-candle pattern detection with state machine
  ✓ Relying on low ADX (consolidation) as primary filter
  ✓ Using volume ratio as secondary confirmation
  ✓ News-aware (blocks entries around major events)
  ✓ Static SL at entry-side band (hard protection)
  ✓ Static TP at opposite band (with optional trailing for profit)

IT IS NOT:
  ✗ A trend-following system (deliberately reverses trends)
  ✗ Using ATR-based stops (uses BB-based stops only)
  ✗ Using trailing stops for loss protection (only for profit)
  ✗ Using dynamic position sizing
  ✗ Using daily loss limits or account risk limits (basic version)
  ✗ Using correlations or hedging strategies
  ✗ Single-symbol only (supports multiple symbols)
  ✗ Single-timeframe only (supports multiple timeframes)

═══════════════════════════════════════════════════════════════════════════════
TESTING CHECKLIST & VALIDATION
═══════════════════════════════════════════════════════════════════════════════

Entry Logic Validation:
  ☐ 1st candle detection works (band touch detection)
  ☐ 2nd candle retreat condition correct (no re-touch)
  ☐ Volume multiplier calculation accurate
  ☐ ADX threshold (25) applied correctly
  ☐ News filter blocks entries correctly
  ☐ 3rd candle entry at correct time (open, not close)
  ☐ Multiple timeframes can run simultaneously
  ☐ Multiple symbols trade independently

Exit Logic Validation:
  ☐ TP at opposite band (correct level)
  ☐ SL at entry-side band (correct level)
  ☐ Both are static (don't change)
  ☐ Exit check runs every candle
  ☐ TP priority if both hit same bar
  ☐ Position closes correctly (full close, no partial)
  ☐ State returns to NO_SIGNAL after exit

State Machine Validation:
  ☐ All state transitions work correctly
  ☐ Timeout logic prevents pattern hangs
  ☐ State persists across candles
  ☐ State resets on trade close
  ☐ News filter blocks at all 3 points
  ☐ Multiple symbols don't interfere

═══════════════════════════════════════════════════════════════════════════════
END OF UPDATED SPECIFICATION - VERSION 2.0
═══════════════════════════════════════════════════════════════════════════════
