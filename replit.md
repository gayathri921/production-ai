# StockAI Pro

## Overview

StockAI Pro is an AI-powered real-time stock trading analytics platform built as a React Native (Expo) mobile/web application with an Express.js backend. It provides market data viewing, portfolio tracking, AI-based stock price predictions, and an AI chat assistant for trading guidance.

The app follows a client-server architecture where the Expo frontend communicates with an Express API server that fetches real-time stock data from Yahoo Finance. Technical analysis (RSI, SMA, EMA, MACD) is computed server-side to generate predictions and trading signals.

**Core Features:**
- **Markets tab**: Browse trending stocks, search for symbols, view real-time quotes
- **Portfolio tab**: Track personal stock holdings with live P&L calculations (stored locally via AsyncStorage)
- **AI Predict tab**: Get AI-generated stock price predictions based on technical indicators
- **Chat tab**: Conversational AI assistant for trading guidance
- **Stock Detail screen**: Interactive charts with multiple time ranges, key stats, and add-to-portfolio functionality

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (Expo / React Native)

- **Framework**: Expo SDK 54 with expo-router for file-based routing
- **Routing**: File-based routing via `expo-router` — tabs live in `app/(tabs)/`, detail screens in `app/stock/[symbol].tsx`
- **State Management**: React Query (`@tanstack/react-query`) for server state; React Context (`PortfolioProvider`) for local portfolio state
- **Local Storage**: `@react-native-async-storage/async-storage` persists portfolio holdings on-device — no server-side portfolio storage
- **UI**: Custom dark theme throughout (defined in `constants/colors.ts`), Inter font family, Ionicons, react-native-svg for charts
- **Platform Support**: iOS, Android, and Web (uses platform-specific adaptations like `KeyboardAwareScrollViewCompat`)
- **Fonts**: Inter (400, 500, 600, 700) loaded via `@expo-google-fonts/inter`

### Backend (Express.js)

- **Runtime**: Node.js with TypeScript (compiled via `tsx` in dev, `esbuild` for production)
- **Server**: Express 5 running on port 5000, serves API routes and optionally static web builds
- **Data Source**: Yahoo Finance via `yahoo-finance2` npm package — provides real-time quotes, historical data, search, and trending stocks
- **Technical Analysis**: Server-side calculations for SMA, EMA, RSI, and MACD — implemented directly in `server/routes.ts` without external ML libraries
- **CORS**: Dynamic CORS configuration supporting Replit dev/deployment domains and localhost for Expo web dev
- **API Routes** (defined in `server/routes.ts`):
  - `GET /api/trending` — trending stock tickers
  - `GET /api/stock/:symbol` — real-time quote for a symbol
  - `GET /api/stock/:symbol/history` — historical price data with configurable range
  - `GET /api/search?q=` — stock symbol search
  - `GET /api/predict/:symbol` — AI prediction based on technical analysis
  - `POST /api/chat` — AI chat endpoint

### Database Schema

- **ORM**: Drizzle ORM with PostgreSQL dialect
- **Schema** (`shared/schema.ts`): Currently defines a `users` table with id, username, and password fields
- **Validation**: `drizzle-zod` generates Zod schemas from Drizzle table definitions
- **Storage**: Currently uses in-memory storage (`MemStorage` in `server/storage.ts`) — the database schema is defined but not actively connected. When Postgres is provisioned, run `npm run db:push` to sync the schema.
- **Config**: `drizzle.config.ts` reads `DATABASE_URL` environment variable

### Build & Deployment

- **Dev workflow**: Two processes run simultaneously — `expo:dev` for the frontend, `server:dev` for the backend
- **Production build**: `expo:static:build` creates a static web bundle, `server:build` bundles the server with esbuild, `server:prod` runs it
- **Static serving**: Production server serves the built Expo web app as static files
- **Build script**: `scripts/build.js` handles the Expo static build process with Metro bundler integration

### Key Design Decisions

1. **Yahoo Finance over paid APIs**: Uses the free `yahoo-finance2` package for market data, avoiding API key requirements. Trade-off: rate limits and potential reliability issues.

2. **Client-side portfolio storage**: Portfolio data is stored in AsyncStorage rather than server-side database. This simplifies the architecture but means portfolio data doesn't sync across devices.

3. **Server-side technical analysis**: All indicator calculations (RSI, SMA, EMA, MACD) happen on the server to keep the mobile client lightweight and ensure consistent results.

4. **In-memory user storage**: The `MemStorage` class stores users in a Map. This is a placeholder — should be migrated to Postgres when the database is provisioned.

5. **Shared schema directory**: `shared/` contains code used by both client and server (Drizzle schema, Zod types), enabling type safety across the stack.

## External Dependencies

### Third-Party Services
- **Yahoo Finance** (`yahoo-finance2`): Real-time stock quotes, historical data, search, and trending stocks. No API key required.

### Database
- **PostgreSQL**: Configured via `DATABASE_URL` environment variable. Schema managed by Drizzle ORM with migrations in `./migrations/`. Use `npm run db:push` to push schema changes.

### Key NPM Packages
- **expo** (~54.0.27): Core framework for cross-platform React Native development
- **expo-router** (~6.0.17): File-based routing
- **express** (^5.0.1): Backend API server
- **drizzle-orm** (^0.39.3): TypeScript ORM for PostgreSQL
- **@tanstack/react-query** (^5.83.0): Server state management and data fetching
- **yahoo-finance2**: Stock market data provider
- **react-native-svg**: Chart rendering
- **react-native-reanimated**: Animations
- **expo-haptics**: Tactile feedback on native platforms

### Environment Variables
- `DATABASE_URL`: PostgreSQL connection string (required for database features)
- `EXPO_PUBLIC_DOMAIN`: Domain for API communication between frontend and backend
- `REPLIT_DEV_DOMAIN`: Replit development domain (auto-set in Replit environment)
- `REPLIT_DOMAINS`: Comma-separated deployment domains for CORS