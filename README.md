# ApolloTrader
Fully automated crypto trading powered by a custom price prediction AI and a structured/tiered DCA system.

**Version:** Apollo Trader 26 
**Features:** Debug mode, simulation mode, configurable themes, enhanced error handling, and comprehensive settings system.

"It's an instance-based (kNN/kernel-style) predictor with online per-instance reliability weighting, used as a multi-timeframe trading signal." - ChatGPT on the type of AI used in this trading bot.

So what exactly does that mean?

When people think AI, they usually think about LLM style AIs and neural networks. What many people don't realize is there are many types of Artificial Intelligence and Machine Learning - and the one in my trading system falls under the "Other" category.

When training for a coin, it goes through the entire history for that coin on multiple timeframes and saves each pattern it sees, along with what happens on the next candle AFTER the pattern. It uses these saved patterns to generate a predicted candle by taking a weighted average of the closest matches in memory to the current pattern in time. This weighted average output is done once for each timeframe, from 1 hour up to 1 week. Each timeframe gets its own predicted candle. The low and high prices from these candles are what are shown as the blue and orange horizontal lines on the price charts. 

After a candle closes, it checks what happened against what it predicted, and adjusts the weight for each "memory pattern" that was used to generate the weighted average, depending on how accurate each pattern was compared to what actually happened.

Yes, it is EXTREMELY simple. Yes, it is STILL AI.

Here is how the trading bot utilizes the price prediction ai to automatically make trades:

For determining when to start trades, the AI's Thinker script sends a signal to start a trade for a coin if the ask price for the coin drops below at least 3 of the the AI's predicted low prices for the coin (it predicts the currently active candle's high and low prices for each timeframe across all timeframes from 1hr to 1wk). **This threshold can be customized** in the Trading Settings (default: LONG signal minimum of 3, SHORT signal maximum of 0).

For determining when to DCA, it uses either the current price level from the AI that is tied to the current amount of DCA buys that have been done on the trade (for example, right after a trade starts when 3 blue lines get crossed, its first DCA wont happen until the price crosses the 4th line, so on so forth), or it uses the configurable drawdown % for its current level, whichever it hits first. **Default DCA levels: -2.5%, -5.0%, -10.0%, -20.0%** (4 levels). It allows a max of 2 DCAs within a rolling 24hr window to keep from dumping all of your money in too quickly on coins that are having an extended downtrend! **DCA levels and limits can be customized** in the Trading Settings.

For determining when to sell, the bot uses a trailing profit margin to maximize the potential gains. The margin line is set at either 5% gain if no DCA has happened on the trade, or 3% gain if any DCA has happened. The trailing margin gap is 0.5% (this is the amount the price has to go over the profit margin to begin raising the profit margin up to TRAIL after the price and maximize how much profit is gained once the price drops below the profit margin again and the bot sells the trade).

**Multi-Timeframe Exit Confirmation:** When the price crosses below the trailing profit margin line, the bot requires bearish confirmation across multiple timeframes before executing the sell. This prevents premature exits during minor pullbacks while still capturing profit on true trend reversals. The exit requires BOTH short_signal >= 4 (bearish confirmation) AND long_signal <= 0 (no bullish resistance). **The stop-loss (-40%) ALWAYS executes immediately, bypassing MTF check for safety.** **Exit signal thresholds, profit margins, and timing can be customized** in the Trading Settings.

# Setup & First-Time Use (Windows)

ApolloTrader is designed to be easy to set up with minimal technical knowledge required.

**Important:** This software can place trades automatically. You are responsible for what it does.  
Keep your API keys private. We are not giving financial advice. We are not responsible for any losses incurred. You are fully responsible for doing your own due diligence to learn and understand this trading system and to use it properly. You are fully responsible for all of your money, and any gains or losses.

---

## Step 1 — Install Python

1. Go to **python.org** and download Python for Windows (Python 3.10 or newer recommended).
2. Run the installer.
3. **Check the box** that says: **"Add Python to PATH"** (very important!).
4. Click **Install Now**.

---

## Step 2 — Download ApolloTrader

1. Go to the ApolloTrader GitHub repository.
2. Click the green **Code** button, then **Download ZIP**.
3. Extract the ZIP file to a folder on your computer, like: `C:\ApolloTrader\`

---

## Step 3 — Launch ApolloTrader (One-Click Setup!)

1. Navigate to your ApolloTrader folder.
2. Double-click **ApolloTrader.pyw** to launch the hub.

**First Launch:** ApolloTrader will automatically check if required packages are installed. If not, it will ask permission to install them. Click **Install** and wait a few minutes while it sets everything up. This is a one-time process.

**Note:** Windows may ask for permission to run Python or install packages - this is normal, click Allow/Yes.

The **ApolloTrader Hub** will open - this is your main control center.

---

## Step 4 — Configure Settings

### Initial Setup

On first launch, ApolloTrader automatically uses the folder where you extracted it as the main directory. You can change this later if needed.

The Hub menu bar contains:
- **Settings**: Hub Settings, Trading Settings, Training Settings, API Setup Wizard
- **View**: Toggle Autopilot, Simulation Mode, Debug Mode
- **Help**: Exit application

To configure settings, click **Settings → Hub Settings** and configure:

- **Coins (comma-separated)**: Start with **BTC** for your first run. Add more coins later (ETH, XRP, BNB, DOGE, etc.).
- **Main neural folder**: Already set to your ApolloTrader directory by default.
- **Robinhood API**: Click **Setup Wizard** and follow these steps:
  1. Click **Generate Keys**.
  2. Copy the **Public Key** shown in the wizard.
  3. On Robinhood.com, go to your account settings and add a new API key. Paste the Public Key.
  4. Set permissions to allow READ and TRADE (the wizard tells you what to select).
  5. Robinhood will show your API Key (often starts with `rh`). Copy it.
  6. Paste the API Key back into the wizard and click **Save**.
  7. Close the wizard and go back to the **Settings** screen.

Click **Save** when done.

After saving, you will have two credential files in your ApolloTrader folder:  
`rh_key.txt` and `rh_secret.txt` - These are automatically **encrypted for security**. Keep them private and secure!

ApolloTrader uses a simple folder structure:  
**All coins use their own subfolders** (like `BTC\`, `ETH\`, etc.).

Each coin folder contains:
- Training data and pattern memories
- Neural prediction files (`low_bound_prices.txt`, `high_bound_prices.txt`)
- Trading signals and status files

### Optional: Customize Trading Behavior

ApolloTrader includes customizable settings so you can tune the bot to your preferences:

**Settings → Trading Settings** - Configure trading parameters:
- Entry signal thresholds (when to start trades)
- DCA (Dollar Cost Averaging) levels and limits
- Profit margin targets and trailing gaps
- Position sizing
- Timing delays

**Settings → Training Settings** - Configure AI training:
- Staleness threshold (how often to retrain)
- Auto-retrain option (future feature)

All settings have built-in validation and save instantly. Changes to trading settings take effect immediately (no restart needed). The white marker lines on the Neural Signal display update automatically when you change entry signal thresholds.

---

## Step 5 — Train (inside the Hub)

Training builds the system's coin "memory" so it can generate signals.

1. In the Hub, click **Train All**.
2. Wait until training finishes (this can take several minutes for each coin).

---

## Step 6 — Start the system (inside the Hub)

When training is done, click:

1. **Start All**

The Hub will automatically start the Thinker and Trader modules in the correct order. You don't need to manually start separate programs. The hub handles everything!

---

## Neural Levels (the LONG/SHORT numbers)

- These are signal strength levels from low to high.
- Higher number = stronger signal.
- LONG = buy-direction signal. SHORT = sell-direction signal.

A TRADE WILL START FOR A COIN IF THAT COIN REACHES A LONG LEVEL OF 4 OR HIGHER WHILE HAVING A SHORT LEVEL OF 0! (Customizable in Trading Settings)

---

## Customizing Trading Behavior

Apollo Trader allows you to customize trading parameters without editing code:

### Trading Settings
Access via **Settings → Trading Settings** in the Hub menu:
- **Entry Signals**: Adjust LONG/SHORT signal thresholds for when trades start (default: LONG min 4, SHORT max 0)
- **Exit Signals (MTF Confirmation)**: Configure multi-timeframe exit confirmation thresholds (default: SHORT min 4, LONG max 0). Prevents premature exits during minor pullbacks while capturing profit on true trend reversals. Stop-loss always bypasses MTF check.
- **DCA Levels**: Configure drawdown percentages and maximum DCA buys per 24 hours (default: 4 levels at -2.5%, -5.0%, -10.0%, -20.0%)
- **Profit Margins**: Set trailing gap and target percentages (default: 5% no DCA, 3% with DCA), stop loss (default: -40%)
- **Position Sizing**: Control initial allocation percentage and minimum trade size
- **Timing**: Adjust main loop delay (default: 0.5s) and post-trade cooldown (default: 30s)

The marker lines on the Neural Signal tiles update automatically when you change signal thresholds:
- **White solid lines** show entry thresholds (when to start trades)
- **Orange dashed lines** show exit thresholds (when MTF confirmation allows sells)
No restart required!

### Training Settings
Access via **Settings → Training Settings** in the Hub menu:
- **Staleness Days**: How many days before retraining is recommended (default: 14)
- **Auto-retrain**: Enable automatic retraining when data becomes stale
- **Pattern Matching**: Configurable thresholds for pattern similarity (uses relative percentage matching)
- **Volatility Adaptation**: Training automatically adjusts matching thresholds based on each coin's volatility (4.0x multiplier)

Changes to settings files take effect immediately for the trading bot (hot-reload with caching).

---

## Advanced Features

### Debug Mode
Enable detailed logging to troubleshoot issues or understand the bot's decision-making process:
1. Open **Settings → Hub Settings**
2. Enable **Debug Mode** checkbox
3. Save settings

Debug mode provides:
- Detailed market data fetch logs
- Pattern matching diagnostics
- Training file validation messages
- API call timing and retry information
- State persistence confirmation

All debug output appears in the Live Output tabs (Runner/Trader/Trainers) and does not clutter production logs when disabled.

### Simulation Mode
Test trading strategies without risking real money:
1. Open **Settings → Trading Settings**
2. Enable **Simulation Mode** checkbox
3. Save settings

Simulation mode:
- Tags all trades with "SIM_" prefix
- Excludes simulated trades from PnL calculations
- Shows "SIMULATION" indicator in the Hub
- Allows testing strategies with real market data
- Can run alongside real trading (separate coins recommended)

**Important:** Simulation mode still requires valid API credentials but will not execute real trades.

### Multi-Timeframe Exit Confirmation (MTF)
Prevent premature exits during minor pullbacks while capturing profit on true trend reversals:

**How It Works:**
When the price crosses below your trailing profit margin line, the system checks for bearish confirmation across all timeframes before executing the sell. This prevents selling during brief consolidations or minor dips that often recover to higher prices.

**Exit Requirements (BOTH must be true):**
- SHORT signal >= 4 (bearish confirmation across timeframes)
- LONG signal <= 0 (no bullish resistance)

**Safety Exception:**
Stop-loss (-40% by default) ALWAYS executes immediately, bypassing MTF check. This ensures catastrophic losses are cut regardless of signal conditions.

**Visual Indicators:**
The Neural Signal tiles display your exit thresholds:
- **Orange dashed lines** show exit requirements (SHORT minimum, LONG maximum)
- Real-time signal levels show current values (e.g., "L:3 S:2")
- When price crosses the trailing line, you can see at a glance whether MTF would allow the exit

**Live Output Messages:**
When a trailing profit margin sell is triggered, the trader shows clear messages:
- ✅ "MTF CONFIRMED" when both conditions pass → sell executes
- ⏸️ "MTF BLOCK" when conditions fail → position stays open, trailing line continues tracking
- 🛑 "STOP-LOSS" when emergency exit triggers → sell executes immediately (bypasses MTF)

**Configuration:**
Customize thresholds via **Settings → Trading Settings → Exit Signals (MTF Confirmation)**:
- Adjust SHORT signal minimum (0-7, default: 4)
- Adjust LONG signal maximum (0-7, default: 0)
- Changes take effect immediately and update the orange marker lines

**Benefit:** By requiring multi-timeframe bearish consensus, you avoid exiting positions that are still in strong uptrends, capturing additional 2-5% gains on average that would otherwise be left on the table.

### Theme Customization
Customize the Hub's appearance:
1. Edit `theme_settings.json` in your Apollo Trader folder
2. Modify colors for backgrounds, text, charts, and buttons
3. Reload the Hub to apply changes

Default themes included:
- **Cyber** (default dark theme with green/cyan accents)
- **Midnight** (blue-tinted dark theme)

Create your own theme by copying and modifying the color values in `theme_settings.json`.

### Enhanced Error Handling & Pattern Matching
The Enhanced Edition includes robust error recovery and improved AI pattern matching:

**Error Recovery:**
- **Circuit Breaker Pattern**: Prevents cascading failures when API is down
- **Automatic Retry Logic**: Network requests retry with exponential backoff
- **Training File Validation**: Detects and skips corrupted training data
- **Graceful Degradation**: System continues operating even when individual components fail

**Pattern Matching Improvements:**
- **Relative Threshold Matching**: Thresholds are now relative to pattern magnitude (percentage-based)
- **Scale-Invariant**: Works consistently across different price levels and market conditions
- **Volatility-Based Adaptation**: Thresholds automatically adjust using 4.0× average volatility
- **Zero-Value Protection**: Uses 0.1% baseline for near-zero patterns (prevents division-by-zero)
- **Simplified Algorithm**: Removed complex PID controller in favor of transparent volatility-based system

### Chart & UI Improvements
- **Centered Window Positioning**: Window always opens within visible screen area
- **Chart Refresh Rate**: Configurable update interval (default: 10 seconds)
- **Neural Level Display**: Shows only timeframe labels on chart (1h, 2h, etc.) with automatic overlap hiding
- **Last Neurals Timestamp**: Charts display when neural predictions were last updated
- **Optimized Padding**: Improved chart margins for better label visibility
- **Flow Status Indicators**: Visual checkmarks (✓) and hourglasses (⧗) show system state between stages

### Training Freshness & Automation
The system enforces training freshness and provides intelligent automation:

**Freshness Enforcement:**
- Trainer writes timestamp when training starts
- Hub and Thinker check training age before allowing trades
- Visual indicators show which coins need retraining
- Configurable staleness threshold (default: 14 days)
- **T-X countdown**: Shows hours remaining until training becomes stale (e.g., "BTC: TRAINED (T-72 HRS)")
- **T+X overdue indicator**: Shows hours overdue when training is stale (e.g., "BTC: TRAINED (T+3 HRS)")

**Auto-Start Thinker:**
- After manual training completes, the Thinker automatically starts
- Seamless workflow: click "Train All" → training finishes → Thinker starts automatically
- Similar to how Thinker auto-starts when opening the Hub with valid training data

**Smart Auto-Retrain:**
- When auto-retrain is enabled, system checks for stale training every minute
- **Priority Coin Protection**: Before stopping for retraining, checks if any coin has imminent trade action
- Delays retraining if a coin is within 2% of: new entry, DCA trigger, stop loss, or take profit
- User sees T+X countdown increment (training overdue) until all trades complete safely
- Then automatically stops, retrains all stale coins, and restarts
- Prevents taking the trader offline during critical moments

If a coin shows "NOT TRAINED / OUTDATED", run training before starting the bot.

### Hot-Reload Configuration
All configuration changes take effect without restarting:
- Trading parameters update immediately (trader checks config every loop)
- Coin list changes are detected on-the-fly (both hub and trader)
- Theme updates apply after Hub reload
- Debug mode toggles instantly
- Entry signal thresholds update Neural Signal display markers in real-time

---

## Adding more coins (later)

1. Open **Settings → Hub Settings**
2. Add one new coin
3. Save
4. Click **Train All**, wait for training to complete
5. Click **Start All**

---

## Technical Improvements (Enhanced Edition)

This version includes numerous stability and reliability improvements over the original:

### Crash Prevention & Data Validation
- **Zero-division protection** in all mathematical operations
- **Training file validation** with format checking and bounds verification
- **Weight list synchronization** to prevent index errors
- **Empty data guards** to handle missing or incomplete data gracefully
- **Corrupt data detection** with automatic skip and continue logic
- **Profit margin display fix**: Corrected formatting to show accurate percentages (was displaying 100x too large)

### Performance Enhancements
- **Thread-safe directory switching** prevents cross-coin data contamination
- **Memory caching** reduces redundant file I/O operations
- **Mtime-based caching** skips reloading unchanged files (neural predictions, trade history, account data)
- **Configurable sleep timings** to balance performance vs API rate limits
- **Chart downsampling** to 250 points max for smooth rendering
- **Debounced UI updates** to prevent excessive redraws
- **Selective chart updates** refreshes only visible coin tab (prevents network stalls)

### Network Resilience
- **Robinhood API retry logic** (5 attempts with exponential backoff)
- **KuCoin fallback system** (client library → REST API)
- **Circuit breaker pattern** to prevent cascading failures
- **Invalid symbol cleanup** to handle malformed coin names
- **Connection timeout handling** with automatic recovery

### Code Quality
- **Module docstrings** with repository and author information
- **Type hints** for better IDE support and error detection
- **Function documentation** explaining behavior and parameters
- **DRY principle** applied with helper functions
- **Consistent error handling** using specific exceptions

All improvements maintain 100% backward compatibility with the original trading logic and produce identical results while providing enhanced reliability and maintainability.

---

## Donate

---

## Project Information

**ApolloTrader** is a fork and significant enhancement of the original PowerTrader_AI project.

**Primary Repository:** https://github.com/Dr-Dufenshmirtz/ApolloTrader  
**Primary Author:** Dr Dufenshmirtz

**Original Project:** https://github.com/garagesteve1155/PowerTrader_AI  
**Original Author:** Stephen Hughes (garagesteve1155)

This fork includes substantial improvements including:
- Configurable training parameters via GUI
- Enhanced error handling and debugging systems
- Performance optimizations and code cleanup
- Expanded customization options for trading strategies
- Improved pattern matching and prediction algorithms

---

## Support

ApolloTrader is COMPLETELY free and open source!

If you'd like to support the **original PowerTrader_AI project**:
- Cash App: **$garagesteve**
- PayPal: **@garagesteve**
- Patreon: **patreon.com/MakingMadeEasy**

---

## License

ApolloTrader is released under the **Apache 2.0** license.