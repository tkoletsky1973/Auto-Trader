BNB-AI-TRADE-ENGINE

SYSTEM OVERVIEW

Execution Model
	•	Dedicated BSC wallet (bot-only)
	•	Private key encrypted server-side
	•	Node.js backend signs transactions automatically
	•	Python AI microservice handles model logic
	•	PostgreSQL for logging + analytics
	•	Web UI dashboard for monitoring + control
	•	Real-time price feeds via WebSocket + reserve polling
	•	Deterministic risk engine (hard stop authority)

⸻

ROOT FOLDER STRUCTURE

bnb-ai-personal-trader/

  docker-compose.yml
  .env.example
  README.md

  frontend/
  backend/
  ai/
  config/
  scripts/
  logs/
  docs/

  ---

  ENVIRONMENT VARIABLES (.env)

  NODE_ENV=production
PORT=4000

BSC_RPC=https://bsc-dataseed.binance.org/
BSC_PRIVATE_RPC=

BOT_PRIVATE_KEY_ENCRYPTED=
ENCRYPTION_SECRET=

GEMINI_API_KEY=
DB_URL=postgresql://user:pass@db:5432/trader

MAX_POSITION_PERCENT=0.20
MAX_DAILY_LOSS_PERCENT=0.05
MAX_SLIPPAGE=0.01
MIN_LIQUIDITY_USD=100000
ARB_PROFIT_BUFFER=0.002

---

DOCKER COMPOSE

version: "3.9"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: trader
      POSTGRES_USER: trader
      POSTGRES_PASSWORD: trader
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    env_file:
      - .env
    depends_on:
      - db
    ports:
      - "4000:4000"

  ai:
    build: ./ai
    env_file:
      - .env
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

---

BACKEND STRUCTURE

backend/
  src/
    app.ts
    server.ts

    config/
    routes/
    controllers/
    services/

      wallet.service.ts
      price.service.ts
      trade.service.ts
      arbitrage.service.ts
      risk.service.ts
      ai.service.ts

    db/
      prisma.schema

WALLET SERVICE (Core Automation)

Responsibilities:
	•	Decrypt private key
	•	Instantiate ethers.js Wallet
	•	Sign transactions automatically
	•	Enforce balance caps

Core Logic:

const wallet = new ethers.Wallet(decryptedKey, provider)

async function executeSwap(txData) {
  const tx = await wallet.sendTransaction(txData)
  return tx.wait()
}

No manual prompts.

PRICE SERVICE

Pulls:
	•	PancakeSwap reserves
	•	BiSwap reserves
	•	Token decimals
	•	Volume estimates

Calculates:
	•	Real-time mid price
	•	Spread delta
	•	Liquidity depth
	•	Slippage projection

Polling every 3 seconds.

⸻

RISK ENGINE (Authority Layer)

Non-negotiable rules:

if positionSize > MAX_POSITION_PERCENT:
    reject

if dailyLoss > MAX_DAILY_LOSS_PERCENT:
    haltBot()

if slippage > MAX_SLIPPAGE:
    reject

if liquidityUSD < MIN_LIQUIDITY_USD:
    reject

Also:
	•	Halt after 3 consecutive losses
	•	Halt if RPC fails 5 times
	•	Halt if price variance > 8% in 1 minute

ARBITRAGE ENGINE

Flow:
	1.	Fetch PCS price
	2.	Fetch BiSwap price
	3.	Calculate spread
	4.	Estimate gas
	5.	Estimate slippage
	6.	Net profit check:

if spread > gas + slippage + ARB_PROFIT_BUFFER:
    executeArb()

Execution:
	•	Swap A → B on lower DEX
	•	Immediately swap B → A on higher DEX

Atomic sequential execution.

AI SERVICE (Python)

ai/
  app.py
  model.py
  features.py
  requirements.txt

  Models:
	•	Gemini Flash 2.5 → reasoning
	•	XGBoost → trade classification

Inputs:
	•	Spread
	•	Volatility
	•	Momentum
	•	Portfolio state
	•	Historical win rate

Output:

{
  pair: "BNB/USDT",
  strategy: "swing",
  position_percent: 0.12,
  stop_threshold: -0.02,
  confidence: 0.81
}

Backend still passes through Risk Engine.

⸻

DATABASE SCHEMA

Tables:

trades
	•	id
	•	timestamp
	•	pair
	•	side
	•	size
	•	entry_price
	•	exit_price
	•	pnl

arb_trades
	•	id
	•	spread
	•	gas_cost
	•	net_profit

risk_events
	•	id
	•	event_type
	•	value
	•	timestamp

ai_logs
	•	id
	•	input_snapshot
	•	output_snapshot
	•	confidence

pnl_snapshots
	•	timestamp
	•	total_balance
	•	daily_pnl

FRONTEND STRUCTURE

frontend/
  pages/
    index.tsx
    dashboard.tsx

  components/
    BalancePanel.tsx
    RiskMeter.tsx
    TradeLog.tsx
    ArbMonitor.tsx
    StrategyPanel.tsx

DASHBOARD LAYOUT

Sections

Top Bar:
	•	Wallet Balance
	•	Unrealized PnL
	•	Daily PnL
	•	Bot Status (ACTIVE / HALTED)

Center:
	•	Live BNB chart
	•	Active trade positions
	•	PCS vs BiSwap spread monitor

Right Panel:
	•	AI Recommendation feed
	•	Auto Mode toggle
	•	Risk Level selector
	•	Position cap slider

Bottom:
	•	Trade history
	•	Arbitrage history
	•	Risk events log

⸻

CONFIG FOLDER

config/
  pairs.json
  blacklist.json
  risk_rules.json

pairs.json example:

[
  "BNB/USDT",
  "BNB/BUSD",
  "ETH/BNB"
]

INSTALLATION PROCESS
	1.	Clone repo
	2.	Add encrypted private key
	3.	Fill .env
	4.	docker-compose up --build
	5.	Fund bot wallet
	6.	Activate bot in dashboard

⸻

TESTING MODULE

Before live mode:
	•	Paper trading mode
	•	Backtest runner
	•	Slippage simulator
	•	Latency simulator

Switchable via:

BOT_MODE=paper

FAILSAFE MECHANISMS
	•	Circuit breaker
	•	Manual kill switch
	•	Auto halt on abnormal volatility
	•	RPC failover
	•	Position size hard caps
	•	Daily drawdown stop
	•	Gas spike filter
	•	Liquidity drop detection

⸻

WHAT IS INTENTIONALLY NOT INCLUDED
	•	No pooled capital
	•	No multi-user logic
	•	No token
	•	No vault factory
	•	No governance
	•	No referral
	•	No NFT

This is a private automated trading engine.

⸻

OPERATIONAL RULES
	•	Keep limited capital in bot wallet
	•	Withdraw profits regularly
	•	Rotate private key quarterly
	•	Use dedicated machine or VPS
	•	Enable server firewall
	•	Do not expose backend publicly without auth
