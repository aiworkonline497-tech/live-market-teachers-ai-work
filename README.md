# Market Teacher Pro 2.0

A static Netlify dashboard with a Netlify Function proxy for Twelve Data and Binance public market data.

## Included
- Modern green/red responsive dashboard.
- Forex: EUR/USD, GBP/USD, USD/JPY, EUR/GBP, AUD/USD, USD/CHF.
- Gold: XAU/USD.
- Crypto: BTC/USD, ETH/USD.
- Timeframes: 1m, 2m, 5m, 10m, 15m, 30m, 1h, 2h, 4h.
- EMA 9/21/50, RSI 14, MACD, ADX, candle structure and volume checks.
- Estimated pressure for Twelve Data instruments; Binance aggressive trade-flow estimate for BTC/ETH.
- Neutral state when evidence is mixed.

## Important limitations
This is a research/educational starter, not a profitable trading system and not financial advice. The displayed model score is a rule-alignment score, NOT a probability or proven accuracy. Backtest and forward-test before risking money.

Forex is decentralized. There is no single consolidated global forex volume or global count of buyers and sellers. Any Twelve Data forex volume field should be treated as provider-specific activity/tick volume. Binance trade flow represents Binance only, not the entire crypto market. Binance secondary flow is recent aggregate-trade data and is not a full order-book imbalance model.

Twelve Data free-plan quotas and symbol entitlements can change. The app uses 1-minute candles to aggregate 2m and 10m. This can consume many API credits; multi-timeframe context also makes extra calls. The refresh interval is 60 seconds by default to reduce quota usage. Check your plan's current API credit limits and data entitlements. If XAU/USD or a symbol is not included in your plan, the function will show the provider error.

## Local run
1. Install Node.js LTS.
2. Extract this folder.
3. Open terminal in this folder and run:
   ```bash
   npm install
   ```
4. Copy `.env.example` to `.env` and add your own Twelve Data key:
   ```env
   TWELVE_DATA_API_KEY=your_real_key
   ```
5. Run:
   ```bash
   npx netlify dev
   ```
6. Open the local URL printed by Netlify CLI (usually http://localhost:8888).

## Deploy to Netlify
1. Create a new GitHub repository and upload the contents of this folder.
2. In Netlify choose **Add new site > Import an existing project** and connect that repository.
3. Build settings:
   - Build command: `npm run build`
   - Publish directory: `.`
   - Functions directory: `netlify/functions` (already in `netlify.toml`)
4. Trigger a fresh deploy.
5. Open your deployed site. Click **API Settings** in the top bar, paste your Twelve Data key, click **Save Key**, then **Test & Load**.
6. The key is stored in this browser's local storage and sent to the Netlify Function in an HTTPS request header. It is not embedded in the public JavaScript bundle or committed to the repository. Each browser/device needs its own key entry. A Netlify environment variable (`TWELVE_DATA_API_KEY`) is still supported as an optional server-side fallback.
7. Test EUR/USD, BTC/USD and XAU/USD. If a symbol fails, check Twelve Data symbol availability for your account.

## Security
Do not paste API keys into `app.js`, `index.html`, GitHub, or public chat. The dashboard's API Settings stores the key in the current browser's local storage and sends it to the Netlify Function over HTTPS. Anyone with access to that browser profile or developer tools may be able to retrieve it; do not use this on a shared/public computer. For a shared production dashboard with many users, use an authenticated backend and server-side secret management instead.

## Signal logic (first version)
Each timeframe evaluates:
- EMA alignment: +1 bullish / -1 bearish / 0 mixed.
- RSI: above 55 bullish / below 45 bearish / otherwise neutral.
- MACD line vs signal.
- Current candle body direction.
- ADX trend-strength gate.
- Volume above recent average plus candle direction.

BULLISH or BEARISH requires at least four non-neutral votes, a net score of at least +3 or -3, and ADX >= 18. Otherwise it reports NEUTRAL. These thresholds are initial heuristics, not validated optimal parameters.

## Recommended next engineering phase
1. Build a historical backtesting engine with spread, fees, slippage and realistic candle-close execution.
2. Record every signal and its later outcome for each symbol/timeframe.
3. Show measured win rate, sample count, expectancy and drawdown only after sufficient observations.
4. Add provider health/rate-limit handling and a proper multi-provider candle comparison.
5. Add a proper Binance order-book depth imbalance panel and WebSocket stream if needed.
